<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-12">
                <div class="card card-default">
                    <div class="card-header">My Shares VUEJS</div>

                    <div class="card-body">

                        <router-link class="btn btn-primary" :to="{ name: 'show', params: {id:'new'}}">Register New Share</router-link>
                        <table class="table">
                            <thead>
                                <tr>
                                    <th>Company</th>
                                    <th>Instrument</th>
                                    <th class="text-right">Qty</th>
                                    <th class="text-right">Price</th>
                                    <th class="text-right">Investment</th>
                                    <th>Trans. date</th>
                                    <th>Actions</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="share in shares" v-bind:key="share.id">
                                    <td>{{ share.company_name }}</td>
                                    <td>
                                        {{ share.instrument_name }}<br>
                                        <small class="badge badge-primary" v-for="tag in share.tags" v-bind:key="tag.id">{{ tag.description}}</small>
                                    </td>
                                    <td class="text-right">{{ share.quantity}}</td>
                                    <td class="text-right">{{ share.price}}</td>
                                    <td class="text-right">{{ share.total_investment}}</td>
                                    <td>{{ share.transaction_date}}</td>
                                    <td>
                                        <button class="btn btn-success btn-small" v-on:click="selectShare(share)" type="button" name="button"><i class="fa fa-edit"></i></button>
                                        <button class="btn btn-danger btn-small" v-on:click="deleteShare(share)" type="button" name="button"><i class="fa fa-trash"></i></button>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                        <b-pagination size="md" v-on:input="getData" :total-rows="pagination.total" v-model="currentPage" :per-page="pagination.per_page"></b-pagination>
                    </div>
                </div>
            </div>
        </div>
    </div>

</template>

<script>

    export default {
        data() {
            return {
                shares:[],
                pagination:{},
                currentPage:1
            }
        },
        created() {
            // Load the paginated list when the object is created

        },
        mounted(){
            this.getData()
        },
        methods: {
            getData() {
                // getting the paginated data from the shares endpoint
                let asd = this
                axios.get('/api/shares?page=' + this.currentPage )
                .then(function(d){
                    asd.shares = d.data.shares.data
                    asd.pagination = d.data.shares

                })
            },

            selectShare(share) {
                // use the router to go to a specific Share
                this.$router.push({ name: 'show', params: { id: share.id }})
            },

            deleteShare(share) {
                let asd = this;

                // show a confirmation notification before we send the delete request
                // we set 2 actions cancel and delete
                this.$toasted.show('Please confirm your intent to delete: ' + share.instrument_name, {
                    icon:'trash',
                    action:[
                        {
                            text:'Cancel',
                            onClick: function(e, toastObject) {
                                toastObject.goAway(0);
                            }
                        },
                        {
                            text:'Delete!',
                            onClick: function(e, toastObject) {
                                toastObject.goAway(0)
                                // send the delete request 
                                axios.delete('/api/shares/' + share.id)
                                .then(function(data){
                                    // on success we just refresh the data
                                    asd.getData();
                                    asd.$toasted.success(
                                        data.data.message,
                                        {
                                            icon:'check',
                                            duration:3000
                                        }
                                    )

                                })
                                .catch(function(error){
                                    toastObject.goAway(0)
                                    // should an error occur we render a nice notification
                                    asd.$toasted.error(
                                        error.response.data.message,
                                        {
                                            icon:'exclamation-circle',
                                            duration:5000
                                        }
                                    )
                                })
                            }
                        }

                    ]
                })
            }
        }
    }
</script>
