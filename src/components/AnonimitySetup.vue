<template>
  <transition name="modal">
    <div class="modal-mask" v-on:click="close" v-show="show">
      <div class="modal-container" @click.stop>
        <!-- Updating -->
        <div v-if="loading" style="text-align:center;width:inherit;display:block;margin:auto;">
          <v-progress-circular indeterminate v-bind:size="100" class="cyan--text"></v-progress-circular>
          <h4>Ocultando...</h4>
        </div>
        <!-- Error -->
        <div v-if="!loading && errorCode" style="text-align:center;width:inherit;display:block;margin:auto;">
          <h4>ERROR {{ errorCode }} - {{ error }}</h4>
        </div>
        <!-- Update Form -->
        <div v-if="!loading && !errorCode">
          <div class="modal-body" style="margin-top:0px;margin-bottom:0px;">
            <h4 class="network-title">Modo Incognito</h4>
            <p class="text-box">
              <b>Modo incognito</b> crea un perfil temporal que puede ser destruido <u><b>SOLO despues de 12 horas</b></u>
              .Todo post o comentario realizado durante y hasta que modo incognito sea desativado,
              <u><b>NO estara</b></u> relacionado a tu perfil original. Este modo no puede ser desactivado hasta
              despues de 12 horas (por seguridad). Para mas informacion  visita la seccion FAQ.
            </p>
            <v-text-field
              v-model="alias"
              label="Nombre de usuario incognito"
              style="margin-bottom:0px;margin-top:0px;padding-bottom:0px;"
              counter
              max="130"
              maxlength="130"
            ></v-text-field>
          </div>
          <div class="modal-footer text-right" style="margin-top:0px;padding-bottoom:0px;">
            <div container fluid>
              <v-layout row wrap>
                <v-flex xs6 style="text-align:left;">
                  <v-btn v-tooltip:top="{ html: 'Cancelar' }"
                  medium class="red white--text" style="margin-left:0px;" @click="close">
                    <v-icon medium dark>cancel</v-icon>
                  </v-btn>
                </v-flex>
                <v-flex xs6 style="text-align:right;">
                  <v-btn v-tooltip:top="{ html: 'Actualizar' }"
                  medium class="primary white--text" style="margin-right:0px;" v-on:click="updateAlias">
                    <v-icon medium dark>autorenew</v-icon>
                  </v-btn>
                </v-flex>
              </v-layout>
            </div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import {standardAuthPut} from '../../utils/maskmob-api'
export default {
  props: ['show', 'user', 'networkName'],
  data () {
    return {
      alias: '',
      loading: false,
      errorCode: '',
      error: ''
    }
  },
  methods: {
    close () {
      this.$emit('close')
      this.text = ''
      this.loading = false
      this.errorCode = ''
      this.error = ''
    },
    async updateAlias () {
      try {
        const response = await standardAuthPut({ 'alias': this.alias }, this.$session.get('JWTOKEN'), '/user/alias')
        if (response.status === 200 && response.data.success) {
          this.$emit('updated')
          this.close()
        } else {
          this.$store.commit('snackbar/push', {
            text: 'Error al actualizar'
          })
        }
      } catch (err) {
        this.$store.commit('snackbar/push', {
          text: 'Error al actualizar'
        })
      }
    }
  },
  mounted () {
    document.addEventListener('keydown', (e) => {
      if (this.show && e.keyCode === 27) {
        this.close()
      }
    })
  }
}
</script>

<style scoped>
* {
    box-sizing: border-box
}

.modal-mask {
    position: fixed;
    z-index: 9998;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, .5);
    transition: opacity .3s ease;
}

.modal-container {
    max-width:45%;
    min-width:300px;
    margin: 40px auto 0;
    padding:0px 30px;
    padding-bottom:20px;
    background-color: #fff;
    border-radius: 2px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
    transition: all .3s ease;
    font-family: Helvetica, Arial, sans-serif;
}

.text-box {
  font-size:120%;
  margin-bottom:0px;
  margin-top:10px;
}

.network-title {
  padding-top:15px;
  margin-bottom:0px;
  padding-bottom:0px;
  margin-left:0px;
  padding-left:0px;
}

/*
 * The following styles are auto-applied to elements with
 * transition="modal" when their visibility is toggled
 * by Vue.js.
 *
 * You can easily play with the modal transition by editing
 * these styles.
 */

.modal-enter {
  opacity: 0;
}

.modal-leave-active {
  opacity: 0;
}

.modal-enter .modal-container,
.modal-leave-active .modal-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}
</style>
