<template>
    <div class="container pt-32">
        <div class="row" v-if="post.subtitle != ''">
            <hr>
            <h2><center><strong>{{ post.subtitle }}</strong></center></h2>
        </div>
        <div class="row" v-if="post.description != '' && post.description != null">
            <hr>
            <h3><center><strong>Descripción: {{ post.description }}</strong></center></h3>
        </div>
        <div v-if="post.iframe != null" class="row">
            <h1><center>{{ post.section_title }}</center></h1>
            <iframe width="600" height="600" :src="`${post.iframe}`" title="" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
        </div>
        <div v-if="check_category_poll == 0">
            <div v-if="poll_question_posts == ''" class="row">
                <div class="col-12" v-for="(post, index) in posts" v-bind:index="index">
                    <router-link @click.native="Track(post.google_tag)" class="boton2" :style="{ background: '#0e385d'}" :to="`/content/show/${post.content_id}`"> 
                        <i v-bind:class="post.icon"></i><br> {{ post.title }}
                    </router-link>
                </div>
            </div>
            <div v-if="poll_question_posts != ''" class="row">
                <div v-if="poll_quantity == 1">
                    <form @submit.prevent="onSubmit" ref="createCollection" enctype="multipart/form-data">
                        <div class="col-12" v-for="(post, index) in poll_question_posts" v-bind:index="index">
                        
                            <h2>{{ post.question }}</h2>
                            <hr>
                            <div class="form-group" v-if="post.answer_type_id == 1">
                                <h4>Selecciona la respuesta marcando en el circulo</h4>
                                <hr>
                                <label class="question_poll_yes_no" style="font-size: 20px;" for="yes">Si</label>   <input style="font-size: 30px !important;" type="radio" sty v-model="form.yes_no_answer[index]" id="yes_no_asnwer" value="Si" required>
                                &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      
                                <label class="question_poll_yes_no" style="font-size: 20px;" for="no">No</label>    <input type="radio" v-model="form.yes_no_answer[index]" id="yes_no_asnwer" value="No" required>

                            </div>
                            <div class="form-group" v-if="post.answer_type_id == 2">
                                <h4>Escriba su respuesta</h4>
                                <hr>
                                <input
                                    type="text" 
                                    v-model="form.text_answer[index]" 
                                    class="form-control"
                                    placeholder="Ingresa la respuesta"
                                    required
                                >
                            </div>

                            <button
                                type="submit" class="btn btn-success btn-icon-split">
                                <span class="icon text-white-50">
                                    <i class="fas fa-check"></i>
                                </span>
                                <span class="text">Guardar</span>
                            </button>
                        </div>
                    </form>
                </div>
                <div v-if="poll_quantity > 1">
                    <div class="col-md-12" v-for="(post, index) in polls" v-bind:index="index">
                        <router-link  class="pollboton" :style="{ background: '#572364'}" :to="`/poll/show/${post.poll_id}`"> 
                            {{ post.title }}
                        </router-link>
                    </div>
                </div>
            </div>
        </div>
        <div v-if="check_category_poll != 0">
            <div class="row">
                <div class="col-12" v-for="(post, index) in posts" v-bind:index="index">
                    <router-link class="boton2" :style="{ background: '#0e385d'}" :to="`/content/show/${post.content_id}`"> 
                        <i v-bind:class="post.icon"></i><br> {{ post.title }}
                    </router-link>
                </div>
            </div>
        </div>
    </div>
	
</template>
<script>
    export default {
        created() {
            this.checkContentPoll();
            this.getPollQuestions();
            this.getPollQuantity();
            this.getPost();
            this.getPosts();
            this.catchUser();
            this.checkDate();
            this.getPolls();
        },
        methods: {
            Track(google_tag) {
                this.$gtag.event('page_view', {
                    page_title: google_tag
                });
            },
            onSubmit(e) {
                this.loading = true;
                e.preventDefault();
                let currentObj = this;
    
                const config = {
                    headers: { 'content-type': 'multipart/form-data' }
                }

                let formData = new FormData();
            
                formData.append('poll_id', this.$route.params.id);
                formData.append('yes_no_answers', this.form.yes_no_answer);

                formData.append('text_answers', this.form.text_answer);

                axios.post('/api/poll/answer', formData, config)
                .then(function (response) {
                    currentObj.success = response.data.success;
                })
                .catch(function (error) {
                    console.log(error);
                })
                .finally(() => {
                    this.loading = false;
                    this.$awn.success("La encuesta han sido contestada", {labels: {success: "Éxito"}});
                    this.$router.push('/');
                });
            },
            checkDate() {
                let formData = new FormData();
                formData.append('page', 'Home');
               
                axios.post('/api/content/date', formData)
                .then(function (response) {
                    currentObj.success = response.data.success;
                })
                .catch(function (error) {
                        console.log(error);
                });
            },
            getPollQuantity() {
                axios.get('/api/poll/quantity/'+ this.$route.params.id)
                .then(response => {
                    this.poll_quantity = response.data.data;
                })
                .catch(function (error) {
                    console.log(error);
                })
                .finally(() => {
                    this.loading = false;
                });
            },
            checkContentPoll() {
                axios.get('/api/category/poll/'+ this.$route.params.id)
                .then(response => {
                    this.check_category_poll = response.data.data;
                })
                .catch(function (error) {
                    console.log(error);
                })
                .finally(() => {
                    this.loading = false;
                });
            },
            getPolls() {
                axios.get('/api/poll/all/'+ this.$route.params.id)
                .then(response => {
                    this.polls = response.data.data;
                })
                .catch(function (error) {
                    console.log(error);
                })
                .finally(() => {
                    this.loading = false;
                });
            },
            getPollQuestions() {
                axios.get('/api/poll/show/'+ this.$route.params.id)
                .then(response => {
                    this.poll_question_posts = response.data.data;
                })
                .catch(function (error) {
                    console.log(error);
                })
                .finally(() => {
                    this.loading = false;
                });
            },
            catchUser() {
                let formData = new FormData();
                formData.append('page', 'category_id_'+this.$route.params.id);
               
                axios.post('/api/user/catch', formData)
                .then(function (response) {
                    currentObj.success = response.data.success;
                })
                .catch(function (error) {
                    console.log(error);
                });
            },
            goHome() {
                this.$router.push('/')
            },
            goBack() {
                this.$router.go(-1)
            },
            getPost() {
                this.loading = true;

                axios.get('/api/category/'+ this.$route.params.id +'/edit')
                .then(response => {
                    this.post = response.data.data;
                })
                .catch(function (error) {
                    console.log(error);
                })
                .finally(() => {
                    this.loading = false;
                });
            },
            getPosts() {
                this.loading = true;

                axios.get('/api/content/show/'+ this.$route.params.id)
                .then(response => {
                    this.posts = response.data.data;
                })
                .catch(function (error) {
                    console.log(error);
                })
                .finally(() => {
                    this.loading = false;
                });
            }
        },
        data: function() {
            return {
                polls: [],
                poll_question_posts: [],
                poll_quantity: 0,
                check_category_poll: '',
                posts: [],
                post: '',
                form: {
                    yes_no_answer: [],
                    text_answer: []
                }
            }
        }
    }
</script>
