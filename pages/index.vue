<template>
    <div>
        <header class="header flex flex-col font-bold justify-start items-center h-[16.65rem] pt-[1.21rem] md:h-[15.5rem]">
            <h1 class="pb-[1.37rem] font-medium text-center text-[1.4rem] text-white md:text-[1.75rem]">IP Address Tracker</h1>

            <!-- Input form -->
            <form
                @submit.prevent="checkForm()"
                class="flex items-center w-full max-w-[18rem] pb-[1.4rem] md: md:max-w-[30.8rem] md:pb-[2.75rem]"
            >
                <input
                    v-model="ip"
                    class="h-[3.2rem] font-normal rounded-l-xl text-gray-mid w-full pl-4"
                    placeholder="Search for any IP address"
                    type="text"
                    value="192.212.174.101"
                />

                <button
                    @click.prevent="checkForm()"
                    class="bg-black rounded-r-xl h-[3.2rem] w-[4rem] flex justify-center items-center"
                    type="submit"
                >
                    <img
                        alt="arrow icon"
                        class="p-4"
                        src="../assets/images/icon-arrow.svg"
                    />
                </button>
            </form>

            <!-- IP info -->
            <main class="bg-white drop-shadow-xl flex items-center justify-center py-[1.25rem] rounded-xl z-[1000] w-full max-w-[18.15rem] md:max-w-[62rem] md:px-[2rem]">
                <img
                    v-if="isLoading"
                    alt="loader"
                    class="animate__animated animate__fadeIn h-[4.5rem]"
                    src="./../assets/images/loader-2.svg"
                >

                <div
                    v-else-if="isError"
                    class="animate__animated animate__fadeIn flex flex-col h-[8rem] text-center justify-center items-center gap-4 md:flex-row md:h-[4.5rem]"
                >
                    <img
                        alt="error"
                        class="w-8"
                        src="./../assets/images/error.png"
                    >
                    <p>Unable to retrieve IP information. Please ensure the IP entered is correct then try again.</p>
                </div>

                <ul
                    v-else
                    class="animate__animated animate__fadeIn flex flex-col gap-4 md:flex-row md:items-between w-full md:justify-between md:my-4"
                >
                    <li
                        v-for="(item, property, index) in ipInfo"
                        :key="`${item}-${index}`"
                        :class="[index !== 3 ? 'md:border-r-[1px] md:border-solid md:border-gray-dark' : '']"
                        class="flex flex-col justify-center gap-1 items-center text-center w-full md:h-[4.5rem] md:justify-start md:items-start md:gap-2"
                    >
                        <p class="uppercase text-gray-dark text-[0.65rem] tracking-widest">{{ property }}</p>
                        <p>{{ item ? item: "-" }}</p>
                    </li>
                </ul>
            </main>
        </header>

        <!-- Leaflet map -->
        <div class="h-[79vh]">
            <client-only>
                <l-map
                    :center="[lat + offset, lng]"
                    :options="{ zoomControl: false }"
                    :zoom="zoom"
                >
                    <l-tile-layer url="http://{s}.tile.osm.org/{z}/{x}/{y}.png"></l-tile-layer>
                    <l-marker :lat-lng="[lat, lng]">
                        <l-icon
                            :iconSize="[40, 50]"
                            icon-url="icon-location.svg"
                        />
                    </l-marker>
                </l-map>
            </client-only>
        </div>


    </div>
</template>

<script>
    import { LMap, LTileLayer, LMarker, LIcon } from 'vue2-leaflet';

    export default {
        name: "index",
        components: {
            LMap,
            LTileLayer,
            LMarker,
            LIcon
        },
        data() {
            return {
                ip: "",
                ipifyData: {},
                isError: false,
                isLoading: false,
                key: "at_zka9S0hqqwWPGRlYmTeLRJo8z9yMI",
                lat: 34.04915,
                lng: -118.09462,
                offset: 0.00015,
                zoom: 17,

            }
        },
        computed: {
            ipInfo() {
                let output = {
                    "ip address" : "",
                    location: "",
                    timezone: "",
                    isp: ""
                };

                if(Object.keys(this.ipifyData).length) {
                    const d = this.ipifyData;

                    output["ip address"] = d.ip;
                    output.location = `${d.location.city}, ${d.location.country} ${d.location.postalCode}`,
                    output.timezone = `UTC ${d.location.timezone}`
                    output.isp = d.isp
                }

                return output;
            }
        },
        methods: {
            checkForm() {
                this.getIp();
            },
            getIp() {
                this.isLoading = true;
                this.isError = false;

                this.$axios.get(`https://geo.ipify.org/api/v2/country,city`, {
                    params: {
                        apiKey: this.key,
                        ipAddress: this.ip
                    }
                }).then(res => {
                    this.lat = res.data.location.lat;
                    this.lng = res.data.location.lng;
                    this.ipifyData = res.data;

                }).catch(err => {
                    console.log("error occured");
                    this.isError = true;

                }).finally(() => {
                    this.isLoading = false;
                });
            },
        },
        // watch: {
        //     ip(newIp, oldIp) {
        //         const regex = /[a-zA-Z]/;

        //         if (regex.test(newIp)) this.ip = oldIp;
        //     }
        // }

    }
</script>

<style lang="scss" scoped>
    .header {
        background-image: url("./assets/images/pattern-bg-mobile.png");
        background-repeat: no-repeat;
        background-size: cover;

        @media screen and (min-width: 768px) {
            background-image: url("./assets/images/pattern-bg-desktop.png");
        }
    }
</style>
