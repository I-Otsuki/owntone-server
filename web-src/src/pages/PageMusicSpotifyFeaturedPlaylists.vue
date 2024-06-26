<template>
  <div class="fd-page-with-tabs">
    <tabs-music />
    <content-with-heading>
      <template #heading-left>
        <p
          class="title is-4"
          v-text="$t('page.spotify.music.featured-playlists')"
        />
      </template>
      <template #content>
        <list-item-playlist-spotify
          v-for="playlist in featured_playlists"
          :key="playlist.id"
          :playlist="playlist"
        >
          <template #actions>
            <a @click.prevent.stop="open_playlist_dialog(playlist)">
              <mdicon
                class="icon has-text-dark"
                name="dots-vertical"
                size="16"
              />
            </a>
          </template>
        </list-item-playlist-spotify>
        <modal-dialog-playlist-spotify
          :show="show_details_modal"
          :playlist="selected_playlist"
          @close="show_details_modal = false"
        />
      </template>
    </content-with-heading>
  </div>
</template>

<script>
import * as types from '@/store/mutation_types'
import ContentWithHeading from '@/templates/ContentWithHeading.vue'
import ListItemPlaylistSpotify from '@/components/ListItemPlaylistSpotify.vue'
import ModalDialogPlaylistSpotify from '@/components/ModalDialogPlaylistSpotify.vue'
import SpotifyWebApi from 'spotify-web-api-js'
import store from '@/store'
import TabsMusic from '@/components/TabsMusic.vue'

const dataObject = {
  load(to) {
    if (store.state.spotify_featured_playlists.length > 0) {
      return Promise.resolve()
    }

    const spotifyApi = new SpotifyWebApi()
    spotifyApi.setAccessToken(store.state.spotify.webapi_token)
    spotifyApi.getFeaturedPlaylists({
      country: store.state.spotify.webapi_country,
      limit: 50
    })
  },

  set(vm, response) {
    if (response) {
      store.commit(types.SPOTIFY_FEATURED_PLAYLISTS, response.playlists.items)
    }
  }
}

export default {
  name: 'PageMusicSpotifyFeaturedPlaylists',
  components: {
    ContentWithHeading,
    ListItemPlaylistSpotify,
    ModalDialogPlaylistSpotify,
    TabsMusic
  },

  beforeRouteEnter(to, from, next) {
    dataObject.load(to).then((response) => {
      next((vm) => dataObject.set(vm, response))
    })
  },
  beforeRouteUpdate(to, from, next) {
    const vm = this
    dataObject.load(to).then((response) => {
      dataObject.set(vm, response)
      next()
    })
  },

  data() {
    return {
      selected_playlist: {},
      show_details_modal: false
    }
  },

  computed: {
    featured_playlists() {
      return this.$store.state.spotify_featured_playlists
    }
  },

  methods: {
    open_playlist_dialog(playlist) {
      this.selected_playlist = playlist
      this.show_details_modal = true
    }
  }
}
</script>

<style></style>
