<template>
    <div>
        <Hero message="Following table contains trade information" title="Trader Dashboard"></Hero>
        <!--SECURITIES-->
        <section class="section">
            <div class="columns">
                <div class="column is-two-thirds">
                    <section class="section" >
                        <h1 class="title" style="text-align: left">Trade Securities
                            <span class="tag" style="font-size:0.5em">Custom Sorted</span>

                        </h1>
                        <b-table
                                :data="securityData"
                                :loading="loading"
                        >

                            <template slot-scope="props">
                                <b-table-column field="isinno" label="ISIN No." sortable>
                                    {{ props.row.isinno }}
                                </b-table-column>
                                <b-table-column field="companyname" label="Company Name" sortable>
                                    {{ props.row.companyname }}
                                </b-table-column>
                                <b-table-column field="sector" label="Sector" sortable>
                                    {{ props.row.sector }}
                                </b-table-column>

                                <b-table-column field="symbol" label="Trade Symbol" sortable>
                                    {{ props.row.symbol }}
                                </b-table-column>
                                <b-table-column field="marketlot" label="Market Lot" sortable>
                                    {{ props.row.marketlot }}
                                </b-table-column>
                                <b-table-column field="pricevariantlimit" label="Price Variant Limit" sortable>
                                    {{ props.row.pricevariantlimit }}
                                </b-table-column>

                                <b-table-column label="Buy Trade">
                                    <a class="tag is-primary" style="font-size:1em"
                                       @click="trade('B',props.row.symbol)">
                                        &nbsp;&nbsp;&nbsp;Buy&nbsp;&nbsp;&nbsp;
                                    </a>
                                </b-table-column>
                                <b-table-column label="Sell Trade">
                                    <a class="tag is-info" style="font-size:1em" @click="trade('S',props.row.symbol)">
                                        &nbsp;&nbsp;&nbsp;Sell&nbsp;&nbsp;&nbsp;
                                    </a>
                                </b-table-column>


                            </template>
                        </b-table>
                    </section>

                    <section class="section" style="margin-top: 2em">
                        <h1 class="title" style="text-align: left">Trade History
                            <span class="tag is-warning" style="font-size:0.5em">Sorted Chronologically</span>

                        </h1>
                        <b-table :data="orderData"  paginated
                                 per-page="6" :columns="orderColumns"></b-table>
                    </section>
                </div>
                <div class="column is-one-thirds">
                    <section class="section">
                        <h1 class="title" style="text-align: left">Your Daily Limits</h1>

                        <div class="notification is-primary">
                            <!--name of sector-->
                            <p class="subtitle" style="text-align: left">{{sectors[0].sector}}
                                <b><i v-if="sectors[1].sectorlimit=='...'" class="fas fa-spinner fa-spin"></i></b>
                            </p>
                            <h1 class="title" style="text-align: left"> &#8377; {{sectors[0].sectorlimit}}</h1>
                        </div>
                        <div class="notification is-primary">
                            <!--name of sector-->
                            <p class="subtitle" style="text-align: left">{{sectors[1].sector}}
                                <b><i v-if="sectors[1].sectorlimit=='...'" class="fas fa-spinner fa-spin"></i></b>
                            </p>
                            <h1 class="title" style="text-align: left"> &#8377; {{sectors[1].sectorlimit}}</h1>
                        </div>


                    </section>

                </div>

            </div>
        </section>    </div>
</template>

<script>
    import Hero from "../Hero";
    import notification from "../../services/notification";
    import axios from "axios";

    export default {
        components: {Hero},
        data() {
            return {

                sectors: [
                    {sector: '...', sectorlimit: '...'},
                    {sector: '...', sectorlimit: ''}
                ],
                loading: false,
                orderData: [],
                orderColumns: [
                    {
                        field: "clientname",
                        label: 'Client Name',
                    },
                    {
                        field: 'security',
                        label: 'Security Name',
                    },
                    {
                        field: 'isinno',
                        label: 'ISIN no.',
                    },
                    {
                        field: 'tradedate',
                        label: 'Trading Data',
                    },
                    {
                        field: 'quantity',
                        label: 'Quantity',
                        centered: true
                    },
                    {
                        field: 'tradetype',
                        label: 'Trade Type',
                    },
                    {
                        field: 'limitprice',
                        label: 'Limit Price',
                    },
                    {
                        field: 'direction',
                        label: "Direction",
                    },
                    {
                        field: 'value',
                        label: "Value",
                    },

                ],
                securityData: [],
                securityColumns: [
                    {
                        field: "companyname",
                        label: 'Company Name',
                    },
                    {
                        field: "sector",
                        label: 'Sector',
                    },
                    {
                        field: "symbol",
                        label: 'Trade Symbol',
                    },
                    {
                        field: "isinno",
                        label: 'ISIN no.',
                    },
                    {
                        field: "marketlot",
                        label: 'Market Lot',
                    },
                    {
                        field: "pricevariantlimit",
                        label: 'Price Variant Limit',
                    },

                ]
            }
        },

        methods: {
            /*
             * Load async data
             */
            trade(direction, symbol) {
                if (direction == "B") {
                    this.$router.push("/orders/" + symbol + "/buy")
                } else
                    this.$router.push("/orders/" + symbol + "/sell")

            },
            loadAsyncData() {
                notification(this, "Fetching data...");


                this.loading = true;
                this.$store.dispatch('getSecurities', this.$store.getters.getUser)
                    .then(data => {
                        console.clear();
                        console.log(data);
                        this.securityData = data;
                    })
                    .catch(error => {
                        console.log(error);
                    });


                this.$store.dispatch('getOrders', this.$store.getters.getUser)
                    .then(data => {
                        this.orderData = data;
                    })
                    .catch(error => {

                    });

                this.loading = false;
            },
        },
        filters: {
            /**
             * Filter to truncate string, accepts a length parameter
             */
            truncate(value, length) {
                return value.length > length
                    ? value.substr(0, length) + '...'
                    : value
            },
        },

        created() {
            this.loadAsyncData();
            //if not loggedin then go back to login page
            if (!this.$store.getters.isLoggedIn) {
                notification(this, "Please login to continue to dashboard..");
                //and go to dashboard
                this.$router.push("/");
            } else {
                console.log("Logged in");
            }

            this.sectors[0].sector = "Updating";
            this.sectors[1].sector = "Updating";

            this.sectors[0].sectorlimit = "...";
            this.sectors[1].sectorlimit = "...";

            setTimeout(() => {
                axios.post("/users/limits", {
                    'brokerid': this.$store.getters.getUser.id
                })
                    .then(({data}) => {
                        this.sectors = data;
                        this.$store.dispatch('updateLimits',data)
                            .then(response=>{
                                console.log("update Limits updated");
                            }).catch(error=>{
                            console.log("ERROR"+error);
                        });
                    })
                    .catch(error => {
                        console.log(error);
                        notification(this, "Error getting sector limits");
                    });

            }, 2000);
        },
        mounted() {
            // setInterval(() => {
            //     this.sectors[0].sector = "Updating";
            //     this.sectors[1].sector = "Updating";
            //
            //     this.sectors[0].sectorlimit = "...";
            //     this.sectors[1].sectorlimit = "...";
            //
            //     setTimeout(() => {
            //         axios.post("/users/limits", {
            //             'brokerid': this.$store.getters.getUser.id
            //         })
            //             .then(({data}) => {
            //                 this.sectors = data;
            //             })
            //             .catch(error => {
            //                 console.log(error);
            //                 notification(this, "Error getting sector limits");
            //             });
            //
            //     }, 2000);
            // }, 2000);

        }
    }

</script>

<style scoped>
    a:hover{
        text-decoration: none;
    }
</style>