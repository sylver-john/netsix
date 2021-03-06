<template>
  <nav class="navbar navbar-toggleable-md navbar-inverse bg-inverse">
    <div class="container">
      <button class="navbar-toggler navbar-toggler-right" type="button" data-toggle="collapse" data-target="#navbarsExampleDefault" aria-controls="navbarsExampleDefault" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>

      <router-link to="/" class="navbar-brand" title="Netsix"><img src="./assets/logo.svg"></router-link>

      <div class="collapse navbar-collapse" id="navbarsExampleDefault">
        <ul class="navbar-nav mr-auto">
          <router-link tag="li" to="/" class="nav-item" exact>
            <a class="nav-link">Home</a>
          </router-link>

          <router-link tag="li" to="/about" class="nav-item" exact>
            <a class="nav-link">About</a>
          </router-link>

          <options></options>

          <li class="nav-item dropdown">
            <a class="nav-link dropdown-toggle" id="dropdown01" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">Help</a>
            <div class="dropdown-menu" aria-labelledby="dropdown01">
              <a class="dropdown-item disabled" href="#">Guide</a>
              <a @click.prevent="closeConnection" class="dropdown-item" href="#">Close connection</a>
              <a v-if="isElectron" @click.prevent="reload" class="dropdown-item" href="#">Refresh components</a>
            </div>
          </li>
        </ul>
        <form class="form-inline my-2 my-lg-0">
          <span v-if="status.isConnected" class="my-2 mr-2 status" title="Connected"></span>
          <span v-if="status.isConnecting" class="my-2 mr-2 status status--pending" title="Connecting"></span>
          <span v-if="!status.isConnected && !status.isConnecting" class="my-2 mr-2 status status--error" title="Disconnected"></span>
          <input :value="useSignaling ? localPeerId : signalingOffer" class="form-control" type="text" placeholder="Generating local ID..." disabled>
          <button v-if="isElectron" class="btn btn-outline-success my-2 my-sm-0 ml-sm-2" type="button" @click="copyToClipboard" :disabled="status.isConnecting">Copy</button>
        </form>
      </div>
    </div>
  </nav>
</template>

<script>
  import { mapState } from 'vuex'
  import { clipboard, remote } from 'electron'
  import Options from './Navbar/Options'

  export default {
    name: 'navbar',
    components: {
      Options
    },
    computed: {
      localId: function () {
        return this.useSignaling ? this.localPeerId : this.signalingOffer
      },
      ...mapState({
        localPeerId: state => state.connection.localPeerId,
        signalingOffer: state => state.connection.signalingOffer,
        useSignaling: state => state.connection.useSignaling,
        status: state => state.connection.status,
        isElectron: state => state.configuration.isElectron
      })
    },
    methods: {
      copyToClipboard () {
        if (this.isElectron) {
          if (!localStorage.getItem('localPeerId')) {
            localStorage.setItem('localPeerId', this.localPeerId)
          }

          clipboard.writeText(this.localId)

          // eslint-disable-next-line no-new
          new Notification('Netsix', {
            body: 'Local ID copied to clipboard!'
          })
        }
      },
      reload () {
        remote.getCurrentWindow().reload()
      },
      closeConnection () {
        window.clientPeer.destroy()
        window.hostPeer.destroy()
      }
    }
  }
</script>

<style scoped>
  img {
    width: 32px; /* 24px */
  }

  .navbar {
    box-shadow: 0 2px 3px rgba(17, 17, 17, .1);
  }

  .status {
    background: rgb(66, 183, 42) none repeat scroll 0 0;
    border-radius: 50%;
    display: inline-block;
    height: 10px;
    margin-left: 4px;
    width: 10px;
  }

  .status--pending {
    background: rgb(243, 156, 18) none repeat scroll 0 0;
  }

  .status--error {
    background: rgb(228, 55, 37) none repeat scroll 0 0;
  }
</style>
