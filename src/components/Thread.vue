<template>
  <div id="threadView">
    <!-- Comment Modal -->
    <commentPostModal :show="showCommentModal" @close="listenToComments();showCommentModal = false" :thread="threadId" @posted="addComment" ></commentPostModal>
    <!-- Loading Div -->
    <div v-if="loading" style="text-align:center;margin-top:30px;width:100%;">
      <v-progress-circular indeterminate v-bind:size="100" class="cyan--text"></v-progress-circular>
      <h4>Cargando...</h4>
    </div>
    <!-- Error Div -->
    <div v-if="error && !loading" style="text-align:center;margin-top:30px;">
      <h4>ERROR {{ errorCode }}</h4>
    </div>
    <v-container v-if="!loading && !error" id="threadContainer" style="padding:5px;">
      <v-layout row wrap justify-center>
        <v-flex xs12 xl9 lg9 md9 offset-md2 offset-xl2 offset-lg2>
          <!-- Thread Container -->
          <v-card>
            <!-- Thread Header -->
            <v-container fluid style="padding:0px;">
              <v-layout row-sm column child-flexl-sm style="padding-top:10px;padding-left:1%;padding-right:1%;">
                <!-- User -->
                <v-flex xs1-12>
                  <div class="text-xs-left">
                    <!-- Easter Egg -->
                    <v-chip v-if="thread.poster.anon && thread.poster.name === 'Dr.Jekyll'">
                      <v-avatar>
                        <img src="../assets/hydeegg.jpg">
                      </v-avatar>
                      <span style="color:DarkRed;">Mr.Hyde</span>
                    </v-chip>
                    <!-- Render for anonymous user -->
                    <v-chip v-else-if="thread.poster.anon">
                      <v-avatar>
                        <img src="../assets/incognito.jpg">
                      </v-avatar>
                      <span style="color:purple;">{{ thread.poster.name }} [anon]</span>
                    </v-chip>
                    <!-- Render for normal user -->
                    <v-chip v-else>
                      <v-avatar>
                        <img :src="thread.poster.thumbnail" style="background-color:grey;" alt="">
                      </v-avatar>
                      <router-link :to="`/profile/${thread.poster.id}`" style="textDecoration:none;">
                        <span class="username-span">{{ thread.poster.name }}</span>
                      </router-link>
                    </v-chip>
                    <v-chip label outline class="hidden-md-and-up grey grey--text" style="float:right;">{{ thread.created_at }}</v-chip>
                  </div>
                </v-flex>
                <!-- Title -->
                <v-flex xs12 style="text-align:center;">
                  <h4 style="margin-bottom:0px;">
                    {{ thread.title }}
                  </h4>
                </v-flex>
                <!-- Date -->
                <v-flex xs1-12 style="text-align:right;" class="hidden-sm-and-down">
                  <v-chip label outline class="grey grey--text">{{ thread.created_at }}</v-chip>
                </v-flex>
              </v-layout>
            </v-container>

            <!-- Thread Content -->
            <v-divider style="margin-top:3px;margin-bottom:5px;"></v-divider>
            <!-- Thread Media and Text Container -->
            <v-container fluid style="padding-top:5px;">
              <v-layout row-sm column child-flex-sm>
                <!-- No media thread -->
                <v-flex v-if="!thread.media" xs12>
                  <p class="text-xs-left thread-text">{{ thread.text }}</p>
                </v-flex>
                <!-- If image media thread -->
                <v-flex v-else-if="thread.media && thread.media.mimetype.match(/image/g)" xs12>
                  <!-- Media Modal -->
                  <imageModal :show="showMediaModal" @close="showMediaModal = false" :source="thread.media.location"></imageModal>
                  <!-- Media Thumbnail -->
                  <v-card-media v-if="thread.text" :src="thread.media.thumbnail" height="200px"
                    class="media-thumbnail" v-on:click="showMediaModal = true">
                    <v-container fluid style="padding:10px;">
                      <v-layout fill-height>
                        <v-flex xs12 align-end flexbox>
                          <v-spacer></v-spacer>
                          <span class="white--text mimetype">
                            <b>{{ thread.media.mimetype }}</b>
                          </span>
                        </v-flex>
                      </v-layout>
                    </v-container>
                  </v-card-media>
                  <v-card-media v-else :src="thread.media.location" height="200px"
                    class="media-thumbnail-big" v-on:click="showMediaModal = true">
                    <v-container fluid style="padding:10px;">
                      <v-layout fill-height>
                        <v-flex xs12 align-end flexbox>
                          <v-spacer></v-spacer>
                          <span class="white--text mimetype">
                            <b>click -> expandir</b>
                          </span>
                        </v-flex>
                      </v-layout>
                    </v-container>
                  </v-card-media>
                  <p class="text-xs-left thread-text">{{ thread.text }}</p>
                </v-flex>
                <!-- If video media thread -->
                <v-flex v-else-if="thread.media && thread.media.mimetype.match(/video/g)" xs12>
                  <div>
                    <video v-if="!thread.text" style="margin:auto;width:50%;display:block;" controls>
                      <source :src="thread.media.location" :type="thread.media.mimetype" />
                    </video>
                    <video v-else style="float:left;width:inherit;max-width:300px;display:block;margin:5px;" controls>
                      <source :src="thread.media.location" :type="thread.media.mimetype" />
                    </video>
                  </div>
                  <p class="text-xs-left thread-text" >{{ thread.text }}</p>
                </v-flex>

              </v-layout>
            </v-container>
          </v-card>

          <!-- Comment Section -->
          <v-container id="commentSection" fluid class="comment-section">
            <v-layout row style="padding:0px;margin:0px;">
              <v-flex xs12 style="padding:0px;margin:0px;">
                <h6 class="text-xs-left" style="margin-bottom:2px;">
                  <v-chip style="margin-left:0px;"><v-icon left small>comment</v-icon>{{ allCommentCount }} COMENTARIOS</v-chip>
                </h6>
              </v-flex>
            </v-layout>
            <v-divider style="margin-top:3px;margin-bottom:5px;"></v-divider>

            <!-- No comments message -->
            <div v-if="commentsOnDisplay.length === 0 && newComments.length === 0" style="text-align:center;margin-top:30px;">
              <h4>NO HAY COMENTARIOS</h4>
            </div>

            <!-- Comment List -->
            <commentComponent v-for="comment in commentsOnDisplay" :key="comment._id" :id="`c${comment._id}`" :commentObj="comment"></commentComponent>
            <v-layout v-if="comments.length !== commentsOnDisplay.length" row class="nopadding-nomargin">
              <v-flex v-if="showAll" xs6-12 class="nopadding-nomargin" style="margin-right:5px;">
                <v-btn block class="grey white--text" v-on:click="loadMoreComments">cargar mas ({{ commentsOnDisplay.length }}/{{ comments.length }})</v-btn>
              </v-flex>
              <v-flex v-if="showAll" xs6-12 class="nopadding-nomargin" style="margin-left:5px;">
                <v-btn block class="grey white--text" v-on:click="loadMoreComments('all')">cargar todos</v-btn>
              </v-flex>
            </v-layout>
            <!-- New Comments list -->
            <commentComponent v-for="comment in newComments" :key="comment._id" :id="`c${comment._id}`" :commentObj="comment"></commentComponent>

          </v-container>
          <!-- Dead Thread Notice -->
          <div v-if="maxCommentsReached" style="text-align:center;margin:30px;">
            <h5>ESTE THREAD HA LLEGADO AL LIMITE DE COMENTARIOS</h5>
          </div>
        </v-flex>
        <v-flex class="hidden-sm-and-down" xs1-2></v-flex>
      </v-layout>
    </v-container>
    <v-btn
    v-if="!maxCommentsReached"
    v-on:click="stopListening();showCommentModal = true"
    style="textDecoration:none;border:0;outline:none;"
    v-tooltip:top="{ html: 'Comentar' }"
    class="orange"
    dark
    fixed
    bottom
    right
    fab
    >
    <v-icon>mode_comment</v-icon>
    </v-btn>
  </div>
</template>

<script>
import {standardAuthGet, standardAuthPost} from '../../utils/maskmob-api'
import commentPostModal from './CommentModal'
import commentComponent from './Comment'
import imageModal from './ImageModal'
const moment = require('moment')
export default {
  name: 'thread',
  data () {
    return {
      // State data
      loading: true,
      errorCode: '',
      error: '',
      thread: null,
      threadId: '',
      showCommentModal: false,
      showMediaModal: false,
      commentsToAdd: 5,
      showAll: true,
      // Comments
      commentsOnDisplay: [],
      comments: [],
      newComments: [],
      // Comment Timer
      commentInterval: null,
      lastLoadedTimestamp: '',
      threadTimestamp: ''
    }
  },
  computed: {
    maxCommentsReached () {
      return (this.comments.length + this.newComments.length) >= 490
    },
    allCommentCount () {
      return this.comments.length + this.newComments.length
    }
  },
  components: {
    commentPostModal: commentPostModal,
    commentComponent: commentComponent,
    imageModal: imageModal
  },
  created () {
    this.threadId = this.$route.params.id
    this.loadThreadInfo(this.threadId)
    this.loadComments(this.threadId)
    this.listenToComments()
  },
  beforeRouteUpdate (to, from, next) {
    console.log(to)
    console.log(from)
    next()
  },
  beforeDestroy () {
    clearInterval(this.commentInterval)
  },
  methods: {
    async loadThreadInfo (threadId) {
      try {
        const response = await standardAuthGet(this.$session.get('JWTOKEN'), `/thread/${threadId}`)
        if (response.status === 200) {
          let thread = response.data.doc
          // Set initial thread timestamp
          this.threadTimestamp = thread.created_at
          thread.created_at = moment(thread.created_at).fromNow()
          this.thread = thread
          this.loading = false
        } else {
          this.errorCode = String(response.status)
          this.error = response.err
          this.loading = false
        }
      } catch (e) {
        this.error = e
        this.loading = false
      }
    },
    async loadComments (threadId) {
      try {
        const response = await standardAuthGet(this.$session.get('JWTOKEN'), `/thread/${threadId}/replies`)
        if (response.status === 200) {
          let commentsResponse = response.data.doc
          if (commentsResponse.length > 0) {
            // Set autoudate timestamp
            this.lastLoadedTimestamp = response.data.doc[response.data.doc.length - 1].created_at
            // Display first n comments
            this.comments = commentsResponse
            this.commentsOnDisplay = this.comments.slice(0, this.commentsToAdd)
          }
        } else {
          this.errorCode = String(response.status)
          this.error = response.err
          this.loading = false
        }
      } catch (e) {
        this.error = e
        this.loading = false
      }
    },
    async loadMoreComments (all) {
      const idx = this.commentsOnDisplay.length
      if (all === 'all') {
        const idxEnd = this.comments.length
        this.commentsOnDisplay.push.apply(this.commentsOnDisplay, this.comments.slice(idx, idxEnd))
        this.showAll = !this.showAll
      } else {
        if (this.comments.length < 50) {
          this.commentsToAdd = 20
        } else if (this.comments.length < 200) {
          this.commentsToAdd = 50
        } else {
          this.commentsToAdd = 100
        }
        this.commentsOnDisplay.push.apply(this.commentsOnDisplay, this.comments.slice(idx, idx + this.commentsToAdd))
      }
    },
    async addComment (comment) {
      // Load comments
      await this.checkForNewComments()
      // Scroll to comment asynchronously
      this.$nextTick(() => document.getElementById(`c${comment._id}`).scrollIntoView())
    },
    async checkForNewComments () {
      try {
        let dt = (this.comments.length === 0 && this.newComments.length === 0) ? new Date(this.threadTimestamp).getTime()
          : new Date(this.lastLoadedTimestamp).getTime()
        const response = await standardAuthPost({ 'date': dt }, this.$session.get('JWTOKEN'),
          `/thread/${this.thread._id}/replies/since`)
        if (response.status === 200 && response.data.success) {
          if (response.data.doc && response.data.doc.length > 0) {
            this.lastLoadedTimestamp = response.data.doc[response.data.doc.length - 1].created_at
            // Notify for new comments
            this.$store.commit('snackbar/push', {
              text: `${response.data.doc.length} nuevos comentarios`
            })
            // Add New Comments
            this.newComments.push.apply(this.newComments, response.data.doc)
            return true
          }
        } else {
          this.$store.commit('snackbar/push', {
            text: 'Revisa tu conexion'
          })
        }
      } catch (err) {
        console.log(err)
        this.$store.commit('snackbar/push', {
          text: 'Revisa tu conexion'
        })
      }
      return false
    },
    stopListening () {
      clearInterval(this.commentInterval)
    },
    listenToComments () {
      this.commentInterval = setInterval(() => {
        this.checkForNewComments()
      }, 48000)
    }
  }
}
</script>

<style scoped>
.user-name {
  font-size: 150%;
  color: green;
  padding-top: 0px;
}
.username-span {
  color:DarkBlue;
  margin-right:5px;
}
.media-thumbnail {
  float: left;
  width: inherit;
  max-width: 300px;
  display: block;
  margin-left:0px;
  margin-bottom: 5px;
  width: 100%;
  display: block;
  margin-right: 5px;
  cursor: pointer;
}

.media-thumbnail-big {
  margin:auto;
  width:50%;
  display:block;
  cursor: pointer;
}

.mimetype {
  background: rgba(0,0,0,0.6);
  padding: 5px;
}
.thread-text {
  font-size: 115%;
  margin: 5px;
  white-space: pre-wrap;
}
.comment-section {
  height:auto;
  background-color:#F0F0F0;
  padding:5px;
}
.nopadding-nomargin {
  margin:0px;
  padding:0px;
}
</style>
