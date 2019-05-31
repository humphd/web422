<template>
    <div class="menu">
        <!--Search Field-->
        <div style="width:100%;" class="ui icon input">
            <input placeholder="Name a bridge..." v-model="search" />
            <i class="search icon"></i>
        </div>
        <div class="menu">
            <!--Loader-->
            <div v-if="status.loading" class="menu ui segment inverted">
                <div class="ui active dimmer">
                    <div class="ui text loader">Loading bridges...</div>
                </div>
            </div>
            <!--Error Modal-->
            <div v-if="status.errored">
                <div class="ui warning message">
                    <div class="header">
                        Error: Data was not loaded.
                    </div>
                    Hint: Check API endpoint!
                </div>
            </div>
            <!--BridgeListing-->
            <div v-else>
                <menu-item
                        v-for="bridge of shownBridges"
                        :key="bridge.id"
                        :bridge="bridge"
                        @click="bridgeSelected"
                />
            </div>
        </div>
    </div>
</template>
<style scoped>
    .menu {
        height: 100%;
    }
</style>
<script>
    import MenuItem from './MenuItem.vue';
    import getBridgeData from '../bridges.js';

    export default {
        name: 'BridgeMenu',
        data: function() {
            return {
                status: {
                    loading: false,
                    errored: false
                },
                search: '',
                bridges: [],
                shownBridges: []
            }
        },
        watch: {
            bridges: function() {
                // Whenever backend API updates
                this.shownBridges = this.bridges;
            },
            search: function() {
                this.shownBridges = this.bridges.filter(bridge =>
                bridge.name.toLowerCase()
                    .includes(this.search.toLowerCase()));
            }
        },
        components: {
            MenuItem
        },
        created: function() {
            this.loadBridges();
        },
        methods: {
            loadBridges: function() {
                this.status.loading = true;

                // Use our bridge.js function to talk to the REST API.
                getBridgeData()
                    .then(bridges => {
                        this.status.loading = false;
                        this.bridges = bridges;
                    })
                    .catch(err => {
                        console.error('Unable to load bridge data', err.message);
                        this.status.errored = true;
                    });
            },
            bridgeSelected: function(bridge) {
                // When the user clicks a menu item, emit a `change`
                // event for the menu control, along with bridge value
                this.$emit('change', bridge);
            }
        }
    }
</script>
