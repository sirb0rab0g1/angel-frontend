<template>
  <v-container fluid fill-height>
    <no-ssr>
      <v-layout wrap justify-center align-center>
        <v-row>
          <v-col
            cols="10"
            lg="9"
            sm="6"
          >
            <v-text-field
              solo
              label="Search Event"
              v-model="search"
              clearable
            ></v-text-field>
          </v-col>
          <v-col
            cols="1"
            lg="1"
            sm="6"
          >
            <v-btn large block @click="searchevent()">
              search
            </v-btn>
            
          </v-col>
          <v-col
            cols="1"
            lg="2"
            sm="6"
          >
            <v-btn large block @click="showeventdialog()">
              Create Event
            </v-btn>
            
          </v-col>


          <!-- table -->
          <v-col
            cols="12"
            lg="12"
            sm="6"
          >
            <v-card>
              <v-simple-table>
              <template v-slot:default>
                <thead>
                  <tr>
                    <th class="text-left">
                      Event Name
                    </th>
                    <th class="text-left">
                      Date
                    </th>
                    <th class="text-left">
                      Summary
                    </th>
                    <th class="text-right">
                      Action
                    </th>
                  </tr>
                </thead>
                <tbody>
                  <tr
                    v-for="(item, index) in requests"
                    :key="index"
                  >
                    <td>{{ item.title }}</td>
                    <td>{{ item.date }}</td>
                    <td>{{ item.summary }}</td>
                    <td class="text-right">
                      <v-btn @click="view(item)">
                        View
                      </v-btn>
                      <v-btn @click="areyousuredeleteevent(item)">
                        Delete
                      </v-btn>
                    </td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>
            </v-card>

            <!-- add event modal -->

            <v-dialog width="900" persistent v-model="showevent">
              <v-card class="pa-4">

                <v-layout row wrap>
                  <v-flex lg12 md12 sm12 xs12>
                    <v-toolbar
                      dark
                      color="primary"
                    >
                      <v-toolbar-title>Manage Event</v-toolbar-title>
                      <v-spacer></v-spacer>
                      <v-toolbar-items>
                        <v-btn
                          icon
                          @click="hideevent()"
                        >
                          <v-icon>mdi-close</v-icon>
                        </v-btn>
                      </v-toolbar-items>
                    </v-toolbar>
                  </v-flex>
                  <v-flex lg6 class="pa-2 mt-2">
                    <v-text-field
                      solo
                      label="Title"
                      v-model="form.title"
                      clearable
                    ></v-text-field>
                    <v-text-field
                      solo
                      label="Date"
                      v-model="form.date"
                      clearable
                    ></v-text-field>
                    <v-textarea
                      solo
                      label="Summary"
                      v-model="form.summary"
                      clearable
                    ></v-textarea>
                  </v-flex>
                  <v-flex lg6 class="pa-2 mt-2">
                    <croppa
                      v-model="croppa"
                      :width="croppa.width"
                      :height="croppa.height"
                      :placeholder="croppa.placeholder"
                      @file-choose="onCropped"
                    >
                      <img slot="initial" :src="form.image" />
                    </croppa>
                  </v-flex>

                  <v-flex lg12>
                    <v-btn @click="createevent()">SAVE</v-btn>
                  </v-flex>
                </v-layout>
              </v-card>
            </v-dialog>
          </v-col>
        </v-row>

        <v-snackbar
          v-model="snackbar"
          timeout="2000"
        >
          {{ text }}

          <template v-slot:action="{ attrs }">
            <v-btn
              color="pink"
              text
              v-bind="attrs"
              @click="snackbar = false"
            >
              Close
            </v-btn>
          </template>
        </v-snackbar>

        <!-- logout dialog -->
        <v-dialog
          v-model="deletedialog"
          persistent
          max-width="400"
        >
          <v-card>
            <v-card-title class="text-h5">
              Are you sure you want to delete event name " {{ choosen.title | capitalizeFirst }} "?
            </v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="green darken-1"
                text
                @click="deletedialog = false"
              >
                Cancel
              </v-btn>
              <v-btn
                color="green darken-1"
                text
                @click="deleteevent(choosen)"
              >
                Proceed
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-layout>
    </no-ssr>
  </v-container>
</template>

<script>
  import { mapGetters } from 'vuex'
  import axios from 'axios'
  import _ from 'lodash'
  import moment from 'moment'
  import io from 'socket.io-client';

  export default {
    data: () => ({
      requests: [],
      search: '',
      showevent: false,
      form: {},
      croppa: {
        width: 400,
        height: 400,
        placeholder: 'Select an image'
      },
      croppedImage: null,
      snackbar: false,
      text: '',
      deletedialog: false,
      choice: null,
      choosen: {}
    }),
    computed: {
      ...mapGetters('users', ['user'])
    },
    methods: {
      showeventdialog () {
        this.$set(this.form, 'image', '')
        this.showevent = true
      },
      async view (payload) {
        await axios.post('http://localhost:5000/search-event', {id: payload.id, title: ''}).then(data => {
          this.showevent = true
          console.log(data.data[0])
          this.form = data.data[0]

          console.log(this.form)
        })
      },
      onCropped(data) {
        this.croppedImage = data
      },
      async searchevent () {
        await axios.post('http://localhost:5000/search-event', {title: _.isNull(this.search) ? '' : this.search, id: null}).then(data => {
          this.requests = data.data
        })
      },
      async createevent () {
        if (_.has(this.form, 'id')) {
          this.deleteevent(this.form)
        } else {
          this.$set(this.form, 'image', '')
        }
        console.log(this.form)
        await axios.post('http://localhost:5000/create-event', this.form).then(data => {
          this.getdataimage(data.data)
        })
      },
      async getallevents () {
        await axios.get('http://localhost:5000/get-all-events').then(data => {
          this.requests = data.data
        })
      },
      async getdataimage (data) {
        const formData = new FormData();
        formData.append('file', this.croppedImage);
        formData.append('eventid', data.id);

        await axios.post('http://localhost:5000/upload', formData)
        .then(data => {
          this.showevent = false
          this.getallevents()
        })
        .catch(error => console.error('Error:', error));
      },
      areyousuredeleteevent(payload) {
        this.deletedialog = true
        this.choosen = payload
        // this.deleteevent(payload)
      },
      async deleteevent (payload) {
        await axios.post('http://localhost:5000/delete-event', payload).then(data => {
          this.snackbar = true
          this.text = data.data.message
          this.deletedialog = false
          this.getallevents()
        })
      },
      hideevent () {
        this.form = {}
        this.$set(this.form, 'image', '')
        this.showevent = false
      }
    },
    mounted () {
      this.getallevents()
    }
  }
</script>