<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-12">
                <div class="card card-default">
                    <div class="card-header">{{ action }} Share VUEJS</div>

                    <div class="card-body" v-if="!editmode">
                        <dl class="row" v-if="share">
                            <dt class="col-sm-3">Company name</dt>
                            <dd class="col-sm-9">{{ share.company_name }}</dd>

                            <dt class="col-sm-3">Instrument name</dt>
                            <dd class="col-sm-9">{{ share.instrument_name }}</dd>
                        </dl>
                        <div class="" v-else>
                            <p>No share selected</p>
                        </div>
                    </div>
                    <div class="card-body" v-if="editmode">

                        <form method="POST" novalidate>
                            <div class="form-group row">
                                <label for="company_name" class="col-sm-4 col-form-label text-md-right">Company Name</label>

                                <div class="col-md-6">
                                    <input id="company_name" type="text" class="form-control" v-model="share.company_name" required autofocus>
                                </div>
                            </div>

                            <div class="form-group row">
                                <label for="instrument_name" class="col-sm-4 col-form-label text-md-right">Instrument Name</label>

                                <div class="col-md-6">
                                    <input id="instrument_name" type="text" class="form-control" v-model="share.instrument_name" required autofocus>
                                </div>
                            </div>

                            <div class="form-group row">
                                <label for="quentity" class="col-sm-4 col-form-label text-md-right">Quantity</label>

                                <div class="col-md-6">
                                    <input id="quantity" type="text" class="form-control" v-model="share.quantity" required autofocus>
                                </div>
                            </div>

                            <div class="form-group row">
                                <label for="price" class="col-sm-4 col-form-label text-md-right">Price</label>

                                <div class="col-md-6">
                                    <input id="price" type="text" class="form-control" v-model="share.price" required autofocus>
                                </div>
                            </div>

                            <div class="form-group row">
                                <label for="certificate_number" class="col-sm-4 col-form-label text-md-right">Certificate Number</label>

                                <div class="col-md-6">
                                    <input id="certificate_number" type="text" class="form-control" v-model="share.certificate_number" required autofocus>
                                </div>
                            </div>

                            <div class="form-group row">
                                <label for="transaction_date" class="col-sm-4 col-form-label text-md-right">Transaction Date</label>

                                <div class="col-md-6">
                                    <input id="transaction_date" type="text" class="form-control" v-model="share.transaction_date" required autofocus>
                                </div>
                            </div>

                            <div class="form-group row">
                                <label for="transaction_date" class="col-sm-4 col-form-label text-md-right">Tags</label>
                                <div class="col-md-6">
                                    <vue-tags-input
                                    v-model="tag"
                                    :tags="tags"
                                    :autocomplete-items="filteredItems"
                                    @tags-changed="a => tags = a"
                                    />
                                </div>
                                
                            </div>

                            <div class="form-group row">
                                <div class="col-md-6 offset-sm-4">
                                    <button v-on:click="saveShare($event)" class="btn btn-primary"><i class="fa fa-save"></i> Save Share</button>
                                    <router-link class="btn btn-light" :to="{ name: 'list'}"><i class="fa fa-clipboard-list"></i> Back to My Shares</router-link>
                                </div>
                            </div>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

    import VueTagsInput from '@johmun/vue-tags-input';

    export default {
        components:{
            VueTagsInput
        },
        data() {
            return {
                share:{
                    company_name:'',
                    instrument_name:'',
                    quantity:'',
                    price:'',
                    certificate_number:''
                },
                companyName:'',
                editmode:true,
                share_id:'',
                action:'Edit',
                tagSelected:[],
                tags:[],
                tag:'',
                autocompleteItems:[]
            }
        },
        created() {
            this.share_id = this.$route.params.id

            if (this.$route.params.id !== 'new') {
                this.show(this.$route.params.id)
            } else {
                this.action = 'Register New'
            }

            this.getTags()
        },
        computed: {
            filteredItems(){
                return this.autocompleteItems.filter(i => {
                    return i.text.toLowerCase().indexOf(this.tag.toLowerCase()) !== -1;
                });
            }
        },
        methods: {
            goBack(e) {
                e.preventDefault();
                this.$router.go(-1);
            },
            getTags() {
                let asd = this
                axios.get('/api/tags/')
                .then(function(d){
                    asd.autocompleteItems = d.data.tags.map(function(a){
                        return {value:a.id, text:a.description}
                    })

                })
            },
            show(id) {
                let asd = this

                axios.get('/api/shares/' + id )
                .then(function(d){
                    asd.share = d.data.share
                    asd.tags = asd.share.tags.map(function(a){
                        return {value:a.id, text:a.description}
                    })



                    delete asd.share.tags;

                })
            },
            saveShare(event) {
                event.preventDefault();
                // set a variable to be able to make calls to the component from inside
                // of the anonymous functions below defined
                let asd = this
                let updated = false;

                let obj = { share:asd.share}

                // taking advantage of laravel's router we can send a POST request
                // with the parameter _method to send a PUT request
                if (this.$route.params.id !== 'new') {
                    obj._method = 'PUT'
                }

                // this ajax call will save the Share data when creating a new record or updating it
                axios.post('/api/shares/' + asd.share.id , obj)
                .then(function(d){
                    // On success it will render a notification for new or updated record
                    if (asd.share.id !== d.data.share.id) {
                        // if it's a new record that have been saved it will navigate to the 
                        // new route givin the id as a parameter so the next time is saved it will update the data
                        // and not make a new record
                        asd.$router.push({name:'show', params: {id:d.data.share.id}});
                        asd.$toasted.success('Share successfuly registered', {duration:3000, icon:'check'})

                    } else {
                        updated = true;
                        asd.$toasted.success('Share successfuly modified', {duration:3000, icon:'check'})
                    }

                    // we refresh the data from the response
                    asd.share = d.data.share

                    // set a new object to send the tags data
                    let objData = {}
                    objData.tags = asd.tags.map(function(a){
                        return {id:a.value, description:a.text}
                    })

                    // even if we are updating the tags we send a post request
                    // because we are syncing the tags we just let laravel do the job
                    axios.post('/api/shares/' + asd.share.id + '/tags', objData)
                    .then(function(d){
                        // asd.$toasted.success('Tags updated propperly', {duration:3000, icon:'check'})
                    })

                    // we can go back if we want
                    // asd.$router.go(-1)
                }).catch(function(error){
                    // Give notifications when errors occur
                    asd.$toasted.error(error.response.data.message, {duration:5000, icon:'atom'});
                    for (let err in error.response.data.errors) {
                        error.response.data.errors[err].map(function(message){
                            asd.$toasted.error(message, {duration:5000, icon:'atom'});
                        });

                    }

                })

            }
        }
    }
</script>
