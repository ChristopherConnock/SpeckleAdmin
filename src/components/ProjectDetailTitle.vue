<template>
  <v-card class='elevation-0 pa-3'>
    <v-layout row wrap>
      <v-flex xs12 class='display-1 font-weight-light text-capitalize my-5'>
        <editable-span v-if='canEdit' :text='project.name' @update='updateName'></editable-span>
        <span v-else>{{project.name}}</span>
      </v-flex>
      <v-flex xs12 class='caption' style='line-height: 32px'>
        <v-icon small>person</v-icon> <span class='caption'>{{project.canRead.length}}</span>&nbsp;
        <v-icon small>import_export</v-icon> <span class='caption'>{{project.streams.length}}</span>&nbsp;
        <v-icon small>fingerprint</v-icon>&nbsp;<strong style="user-select:all">{{project._id}}</strong>&nbsp;
        <v-icon small>access_time</v-icon>&nbsp;<timeago :datetime='project.updatedAt'></timeago>&nbsp;
        <span class='caption font-weight-light text-uppercase'>Owned by <strong>{{owner}}</strong></span>
      </v-flex>
      <v-flex xs12 class='ma-0 pa-0 mb-2'>
        <v-combobox :menu-props='{"maxHeight":0, "zIndex":"0"}' @input='updateTags' md-disabled='!canEdit' v-model="project.tags" :items='project.tags' hint='add or remove tags' solo persistent-hint small-chips deletable-chips multiple tags>
          <template v-slot:no-data>project has no tags.</template>
        </v-combobox>
      </v-flex>
    </v-layout>
  </v-card>
</template>
<script>
import debounce from 'lodash.debounce'
import uniq from 'lodash.uniq'

export default {
  name: 'HelloWorld',
  props: {
    project: Object
  },
  computed: {
    canEdit( ) {
      return this.project.owner === this.$store.state.user._id || this.project.canWrite.indexOf( this.$store.state.user._id ) > -1
    },
    owner( ) {
      let u = this.$store.state.users.find( user => user._id === this.project.owner )
      if ( !u ) {
        this.$store.dispatch( 'getUser', { _id: this.project.owner } )
      }
      return u ? u.surname.includes( "is you" ) ? `you` : `${u.name} ${u.surname}` : 'Loading'
    },
    canReadProject( ) { return this.project.canRead },
    canWriteProject( ) { return this.project.canWrite },
    canReadStreams( ) { return this.project.permissions.canRead },
    canWriteStreams( ) { return this.project.permissions.canWrite },
    allUsers( ) {
      return uniq( [ ...this.canReadProject, ...this.canWriteProject, ...this.canReadStreams, ...this.canWriteProject, this.project.owner ] )
    }
  },
  data( ) { return {} },
  methods: {
    updateName( args ) {
      this.$store.dispatch( 'updateProject', { _id: this.project._id, name: args.text } )
    },
    updateTags: debounce( function( e ) {
      this.$store.dispatch( 'updateProject', { _id: this.project._id, tags: this.project.tags } )
    }, 1000 ),
  }
}

</script>
<style scoped lang='scss'>
.title-card {
  margin-left: 0;
  margin-right: 0;
}

.project-name {
  transition: all 0.2s ease;
}

.project-name:hover {
  color: #448aff;
}

</style>
