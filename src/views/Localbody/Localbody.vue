<template>
    <div>
        <base-header type="gradient-success" class="pb-6 pb-8 pt-5 pt-md-8">
        </base-header>
        <div class="container-fluid mt--7">
            <div class="row">
                <div class="col-12">
                    <div class="card shadow">
                        <div  class="card-header d-flex justify-content-between flex-column flex-md-row align-items-center">
                            <h3>Localbodies</h3>
                            <div class="d-flex align-items-center justify-content-around flex-column flex-md-row">
                                <!-- FILTER BY DISTRICT -->
                                <base-button size="sm" v-if="pageLoading"><i class="ni ni-settings-gear-65 spin"></i></base-button>
                                <base-dropdown v-else position="right" class="mb-2 mb-md-0">
                                    <base-button slot="title" type="primary" class="dropdown-toggle text-capitalize" size="sm">
                                        {{ selectedDistrict || 'District' }}
                                    </base-button>
                                    <a class="dropdown-item text-black"
                                        @click="selectedDistrict = null"
                                    >
                                        All
                                    </a>
                                    <a class="dropdown-item text-black text-capitalize"
                                        v-for="(item, index) in districts"
                                        :key="index"
                                        @click="selectedDistrict = item.name"
                                    >
                                        {{ item.name }}
                                    </a>
                                </base-dropdown>
                            </div>
                        </div> <!-- Outer Header -->
                        <div 
                            class="card-body table-responsive position-relative p-0 custom__scrollbar"
                            :class="[{'overflow-hidden': pageLoading}]"
                        >
                            <base-table
                                :data="localbodies"
                                type="hover table-striped table-sm"
                            >
                                <template slot="columns">
                                    <th class="text-left">Name</th>
                                    <th>District</th>
                                    <th>Email</th>
                                    <th>Phone</th>
                                    <th>Total wards</th>
                                    <th>Type</th>
                                    <th></th>
                                </template>

                                <template slot-scope="{row}">
                                    <td class="text-left text-capitalize">
                                        {{ row.name || 'N/A' }}
                                    </td>
                                    <td class="text-capitalize">
                                        {{ row.district || 'N/A' }}
                                    </td>
                                    <td>
                                        {{ row.email || 'N/A' }}
                                    </td>
                                    <td>
                                        <div>
                                            <strong>General:</strong> {{ row.phone || 'N/A' }}
                                        </div>
                                        <div>
                                            <strong>Emergency:</strong> {{ row.emergency_phone || 'N/A' }}
                                        </div>
                                        <div>
                                            <strong>Kitchen:</strong> {{ row.kitchen_phone || 'N/A' }}
                                        </div>
                                    </td>
                                    <td>
                                        {{ row.total_wards || 'N/A' }}
                                    </td>
                                    <td class="text-capitalize">
                                        {{ row.type || 'N/A' }}
                                    </td>
                                    <td>
                                        <base-button
                                            v-if="!storeLoading"
                                            icon="fa fa-edit"
                                            size="sm"
                                            type="primary"
                                            title="Edit localbody."
                                            @click="editLocalbody(row.localbody_id)"
                                        ></base-button>
                                        <base-button
                                            v-if="!row.store_id && !storeLoading"
                                            icon="fa fa-store"
                                            size="sm"
                                            type="success"
                                            title="Create seperate store for this localbody."
                                            @click="addStore(row.localbody_id)"
                                        ></base-button>
                                        <loading v-if="storeLoading === row.localbody_id" size="sm"/>
                                    </td>
                                </template>
                            </base-table> <!-- Table -->
                            <div class="over__lay d-flex align-items-center" v-if="pageLoading">
                                <loading color="dark"/>
                            </div>
                        </div> <!-- card body -->
                        <div class="card-footer">
                            <div class="d-flex justify-content-end mb-3">
                                <base-button type="success" @click="addModal = true">
                                    <font-awesome-icon icon="plus" class="mr-2"/>
                                    Create Localbody
                                </base-button>
                            </div>
                            <base-pagination
                                :page-count="total_pages"
                                v-model="page"
                                align="center">
                            </base-pagination>
                        </div> <!-- card footer -->
                    </div> <!-- Outer Card -->
                </div>
            </div>
        </div>
        <!-- ADD LOCALBODY MODAL -->
        <modal :show.sync="addModal" modalClasses="modal-dialog-scrollable" :clickOut="false">
            <template slot="header">
                <h1 class="modal-title">Add Localbody</h1>
            </template>
            <div class="container">
                <add-localbody :key="Date.now()"
                    @close="closeModal"
                    :districts.sync="districts"
                ></add-localbody>
            </div>
        </modal>
        <!-- EDIT LOCALBODY MODAL -->
        <modal :show.sync="editModal" modalClasses="modal-dialog-scrollable" :clickOut="false">
            <template slot="header">
                <h1 class="modal-title">Edit Localbody</h1>
            </template>
            <div class="container">
                <edit-localbody :key="Date.now()"
                    @close="closeModal"
                    :localbodyId.sync="editId"
                    :districts.sync="districts"
                ></edit-localbody>
            </div>
        </modal>
    </div>
</template>
<script>
import AddLocalbody from './AddLocalbody';
import EditLocalbody from './EditLocalbody';

export default {
    name: 'localbody',
    components: {
        AddLocalbody,
        EditLocalbody,
    },
    data: () => ({
        page: 1,
        per_page: 10,
        count: 0,
        total_pages: 0,
        pageLoading: null,
        addModal: false,
        localbodies: [],
        districts: [],
        selectedDistrict: null,
        storeLoading: null,
        editModal: false,
        editId: null,
    }),
    watch: {
        page() {
            this.refreshPage();
        },
        selectedDistrict() {
            this.refreshPage();
        }
    },
    methods: {
        closeModal() {
            this.addModal = false;
            this.editModal = false;
            this.refreshPage()
        },
        editLocalbody(localbody_id) {
            this.editModal = true;
            this.editId = localbody_id;
        },
        listLocalbodies() {
            this.pageLoading = true;
            const page = this.page;
            const per_page = this.per_page;
            const district = this.selectedDistrict;

            this.$axios({
                method: 'get',
                url: '/localbodies/list',
                params: {
                    page,
                    per_page,
                    district
                },
            }).then((response) => {
                const localbodies = response.data.localbodies;

                this.localbodies = localbodies.rows;
                this.count = localbodies.count;
                this.total_pages = Math.ceil( this.count/this.per_page );
            }).finally(() => {
                this.pageLoading = false;
            });
        },
        listDistricts() {
            this.$axios({
                method: 'get',
                url: '/localbodies/districts',
            }).then((response) => {
                const districts = response.data.districts;

                this.districts = districts.rows;
            });
        },
        refreshPage() {
            this.listLocalbodies();
        },
        addStore(localbody_id) {
            this.storeLoading = localbody_id;

            this.$axios({
                method: 'post',
                url: '/localbodies/store/add',
                data: {
                    localbody_id,
                }
            }).then((response) => {
                if(response.data && response.data.status === 'success'){
                    this.$success('Store created.');
                } else {
                    throw new Error('Store not created');
                }
            }).catch(() => {
                this.$error('Store not created.');
            }).finally(() => {
                this.storeLoading = null;
                this.refreshPage();
            });
        }
    },
    mounted() {
        this.listLocalbodies();
        this.listDistricts();
    }
};
</script>
<style scoped>
th, td {
    text-align: center;
    max-width: 200px;
    overflow: auto;
}
</style>
