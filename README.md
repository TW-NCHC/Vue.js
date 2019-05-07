# Sample code for Vue.js


```
<template>
    <div>
        <v-layout row wrap>
            <v-flex xs2>
                <Menu />
            </v-flex>
            <v-flex xs10>
                <v-container grid-list-md style="max-width:100%">
                    <Title
                        :breadcrumbs="breadcrumbs"
                        :title="$t('$K8S.Overview')"
                    />
                    <v-layout row wrap class="justify-end">
                        <v-flex xs4>
                            <v-btn
                                large
                                block
                                dark
                                @click="submit()"
                                color="primary"
                                name="landingpage_vm_submit"
                            >
                                {{ $t('LandingPage.$K8S.Start') }}
                            </v-btn>
                        </v-flex>
                    </v-layout>
                    <v-layout row wrap>
                        <OverviewCard
                            :subject="$t('Dashboard.ComputingVPCC')"
                            :items="[$K8Ses]"
                            :url="['/$USER/$K8S/solutions/list']"
                            :title="[$t('LandingPage.$K8S.Title')]"
                        />
                    </v-layout>
                </v-container>
            </v-flex>
        </v-layout>
    </div>
</template>

<script>
    const config = require('config');
    import { mapGetters, mapActions } from 'vuex';
    import Menu from '$ROOT_DIR/$USER/VM/Menu.vue';
    import GreyCircularBtn from '$ROOT_DIR/common/GreyCircularBtn.vue';
    import OverviewCard from '$ROOT_DIR/common/OverviewCard.vue';
    import Title from '$ROOT_DIR/common/TitleComponent.vue';

    export default {
        name: '$K8S',
        data () {
            return {}
        },
        created: async function() {
            this.initBreadcrumbs();
            await this.fetchDashboard();
        },
        components: {
            Menu,
            GreyCircularBtn,
            OverviewCard,
            Title,
        },
        methods: {
            fetchDashboard: async function() {
                this.$store.commit('Platform_Reset');
                this.$store.commit('$K8S_Reset_Dashboard');

                await this.$store.dispatch('Platform_GetItems_All', { // get platforms, platform, host
                    apiKey: this.apiKey
                });

                // fetch openstack data
                this.allPlatforms['openstack'].map(async (element, index) => {
                    let project = await this.$store.dispatch('Project_Platform_GetId', { // get project id
                        platform: element.group,
                        host: element.group,
                        projectName: this.projectName,
                        apiKey: this.apiKey
                    });

                    // fetch $K8S
                    this.$store.dispatch('$K8S_List_GetItems', {
                        projectid: project.id,
                        platform: element.group,
                        host: element.group,
                        apiKey: this.apiKey,
                        all_$USERs: this.permission[this.projectName] ? 1 : 0,
                        onlyCount: true,
                    });
                })

                await this.$store.dispatch('Platform_GetItems', { // get platforms, platform, host
                    type: 'openstack',
                    apiKey: this.apiKey
                });
            },
            initBreadcrumbs () {
                this.breadcrumbs = [
                    {
                        text: this.$t('Breadcrumbs.Title'),
                        disabled: false,
                        action: () => {
                            this.$router.push('/$USER/dashboard');
                        }
                    },
                    {
                        text: this.$t('$K8S.Overview'),
                        disabled: true,
                        action: () => {

                        }
                    }
                ]
            },
            submit () {
                this.$router.push('/$USER/$K8S/solutions/list');
            },
            toLink (url, platform) {
                if (!url)
                    return;

                if (platform)
                    sessionStorage.setItem('openstack', platform);

                this.$router.push(url);
            }
        },
        watch: {
            projectName: async function(newVal, oldVal) {
                if (oldVal) // skip first time
                    await this.fetchDashboard();
            },
            language (newVal, oldVal) {
                this.initBreadcrumbs();
            }
        },
        computed: mapGetters({
            $USERInfo :'Project_Get$USER_Info',
            wallet: 'Project_GetWallet_Info',
            apiKey: 'Project_GetProject_API_Key',
            projectName: 'Project_NCHC_GetProject_Name',
            permission: 'Project_GetPermission',
            allPlatforms: 'Platform_GetItems_All',
            $K8Ses: '$K8SDashboard_Data',
            language :'Project_GetLanguage'
        })
    }
</script>
```
