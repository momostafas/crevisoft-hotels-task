<template>
<v-layout>
    <v-container fluid>
        <v-card class="mb-6 pb-4">
            <v-system-bar color="#414141" dark :window="true">
                <v-icon large>fas fa-ellipsis-h</v-icon>
                <v-spacer></v-spacer>
                <span>Search Hotel</span>
                <v-spacer></v-spacer>
            </v-system-bar>
            <v-layout class="pt-4" align-center justify-space-around row>
                <v-flex xs12 md3>
                    <v-menu v-model="From" :close-on-content-click="false" :nudge-right="40" transition="scale-transition" offset-y full-width min-width="290px">
                        <template v-slot:activator="{ on }">
                            <v-text-field v-model="fromDate" label="From" prepend-icon="event" readonly v-on="on"></v-text-field>
                        </template>
                        <v-date-picker v-model="fromDate" @input="From = false"></v-date-picker>
                    </v-menu>
                </v-flex>
                <v-flex xs12 md3>
                    <v-menu v-model="To" :close-on-content-click="false" :nudge-right="40" transition="scale-transition" offset-y full-width min-width="290px">
                        <template v-slot:activator="{ on }">
                            <v-text-field v-model="toDate" label="To" prepend-icon="event" readonly v-on="on"></v-text-field>
                        </template>
                        <v-date-picker v-model="toDate" @input="To = false"></v-date-picker>
                    </v-menu>
                </v-flex>
                <v-flex xs12 md3>
                    <v-btn @click="filterDates">Search</v-btn>
                </v-flex>
            </v-layout>
            <v-spacer></v-spacer>
        </v-card>
        <v-card class="mob-6" v-show="showHotels">
            <v-system-bar color="#414141" dark :window="true">
                <v-icon large>fas fa-ellipsis-h</v-icon>
                <v-spacer></v-spacer>
                <span>Hotel Listing</span>
                <v-spacer></v-spacer>
            </v-system-bar>
            <v-layout class="pt-8" align-center justify-space-around row>
                <v-flex xs12 md3>
                </v-flex>
                <v-flex xs12 md8>
                    <v-layout align-center justify-space-around row>
                        <v-flex xs12 md5>
                            <p class="font-weight-bold">Total Nights: <span class="font-weight-light">{{nights}}</span></p>
                        </v-flex>
                        <v-flex xs12 md5>
                            <v-layout>
                                <v-flex md5>
                                    <v-btn @click="sortByPrice">Sort By Price</v-btn>
                                </v-flex>
                                <v-flex md5>
                                    <v-btn @click="sortByName">Sort By Name</v-btn>
                                </v-flex>
                            </v-layout>
                        </v-flex>
                    </v-layout>
                </v-flex>
            </v-layout>
            <v-layout align-center justify-space-around row>
                <v-flex xs12 md3 style="align-self: flex-start;">
                    <v-text-field v-model="search" @click:append="filterHotels" @keyup.enter.native="filterHotels" append-icon="search" label="Search" outlined rounded>
                    </v-text-field>
                    <v-slider thumb-label="always" @change="filterPrice" :min="minPrice" v-model="priceFilter" :max="maxPrice" label="Price Filter"></v-slider>
                </v-flex>
                <v-flex xs12 md8>
                    <v-layout align-center justify-space-around row>
                        <v-flex xs12 md5 v-for="(hotel, hotelIndex) in hotelsListFiltered" v-show="hotel.available" :key="hotelIndex">
                            <v-card outlined class="mb-3 mt-3">
                                <v-list-item three-line>
                                    <v-list-item-content>
                                        <v-list-item class="font-weight-bold">Name: <span class="font-weight-light">
                                                {{hotel.name}} </span></v-list-item>
                                        <v-list-item class="font-weight-bold">Price: <span class="font-weight-light">
                                                {{hotel.price * nights | roundPrice}} AED </span></v-list-item>
                                        <v-list-item class="font-weight-bold">City: <span class="font-weight-light">
                                                {{hotel.city}} </span></v-list-item>
                                    </v-list-item-content>
                                </v-list-item>
                            </v-card>
                        </v-flex>
                        <v-flex xs12 md5></v-flex>
                    </v-layout>
                </v-flex>
            </v-layout>
        </v-card>
    </v-container>
</v-layout>
</template>

<script>
import axios from 'axios';
import Moment from 'moment';
import {
    extendMoment
} from 'moment-range';
const moment = extendMoment(Moment);
export default {
    data() {
        return {
            hotelsList: [],
            hotelsListFiltered: [],
            fromDate:new Date().toISOString().substr(0, 10),
            toDate: new Date().toISOString().substr(0, 10),
            showHotels: false,
            From: false,
            To: false,
            search: '',
            nights: "",
            priceFilter: 0
        };
    },
    methods: {
        // Get all hotels 
        getHotels() {
            axios.get("https://api.myjson.com/bins/tl0bp").then(res => {
                this.hotelsList = res.data.hotels;
                this.hotelsListFiltered = res.data.hotels;
            });
        },
        filterDates() {
            this.showHotels = false;
            let from = moment(this.fromDate);
            let to = moment(this.toDate); // Format Both Hotels for moment to be able to get time diffrence
            let timeDiff = to.diff(from); // Get total time diffrence 
            let days = Math.ceil(timeDiff / (1000 * 60 * 60 * 24)) // get total days diffrence
            this.nights = days;
            this.hotelsListFiltered.forEach(hotel => {
                hotel.available = false; //set available key to defualt false to all hotels
                hotel.availability.forEach(days => {
                    let formatedFrom = moment(days.from, 'DD-MM-YYYY').format('YYYY-MM-DD');
                    let formatedTo = moment(days.to, 'DD-MM-YYYY').format('YYYY-MM-DD'); //format date from api to moment to match the one from date-picker
                    let range = range = moment().range(formatedFrom, formatedTo);
                    let validFrom = range.contains(from);
                    let validTo = range.contains(to)
                    if (validFrom && validTo) {
                        hotel.available = true; // set hotel to valid and show it if it has valid time
                    }
                })
            })
            this.showHotels = true;
        },
        filterHotels() {
            this.hotelsListFiltered = this.hotelsList.filter(hotel => hotel.name.toLowerCase().indexOf(this.search.toLowerCase()) !== -1);
        },
        filterPrice(){
            this.hotelsListFiltered = this.hotelsList.filter(hotel => hotel.price * this.nights <= this.priceFilter );
        },
        sortByName(){
            this.hotelsListFiltered = this.hotelsListFiltered.sort((a, b) => (a.name > b.name) ? 1 : -1)
        },
        sortByPrice(){
            this.hotelsListFiltered = this.hotelsListFiltered.sort((a, b) => (a.price > b.price) ? 1 : -1)
        }
    },
    mounted() {
        this.getHotels();
    },
    computed: {
        minPrice() {
            let min = undefined;
            this.hotelsList.forEach((hotel, hotelIndex) => {
                if(hotelIndex == 0){
                    min = hotel.price * this.nights;
                }
                else if(hotel.price * this.nights < min) {
                    min = hotel.price * this.nights
                }
            })
            return min;
        },
        maxPrice() {
            let max = 0;
            this.hotelsList.forEach(hotel => {
                if (hotel.price * this.nights > max) {
                    max = hotel.price * this.nights
                }
            })
            // this.priceFilter = max; // set defualt value of price filter to Max price
            return max;
        }
    },
    filters: {
        roundPrice(price) {
            return Math.round(price * 100) / 100; // round total price
        }
    }
};
</script>
