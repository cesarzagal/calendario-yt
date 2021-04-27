<template>
  <v-row class="fill-height">
    <v-col>
      <v-sheet height="64">
        <v-toolbar
            flat
        >
          <v-btn color="primary" dark class="mr-1" @click="dialog = true">
            Agregar
            <v-icon small>
              mdi-plus
            </v-icon>
          </v-btn>
          <v-btn
              outlined
              class="mr-4"
              color="grey darken-2"
              @click="setToday"
          >
            Hoy
          </v-btn>
          <v-btn
              fab
              text
              small
              color="grey darken-2"
              @click="prev"
          >
            <v-icon small>
              mdi-chevron-left
            </v-icon>
          </v-btn>
          <v-btn
              fab
              text
              small
              color="grey darken-2"
              @click="next"
          >
            <v-icon small>
              mdi-chevron-right
            </v-icon>
          </v-btn>
          <v-toolbar-title v-if="$refs.calendar">
            {{ $refs.calendar.title }}
          </v-toolbar-title>
          <v-spacer></v-spacer>
          <v-menu
              bottom
              right
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                  outlined
                  color="grey darken-2"
                  v-bind="attrs"
                  v-on="on"
              >
                <span>{{ typeToLabel[type] }}</span>
                <v-icon right>
                  mdi-menu-down
                </v-icon>
              </v-btn>
            </template>
            <v-list>
              <v-list-item @click="type = 'day'">
                <v-list-item-title>Día</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = 'week'">
                <v-list-item-title>Semanal</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = 'month'">
                <v-list-item-title>Mensual</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = '4day'">
                <v-list-item-title>4 días</v-list-item-title>
              </v-list-item>
            </v-list>
          </v-menu>
        </v-toolbar>
      </v-sheet>
      <v-sheet height="600">
        <v-calendar
            ref="calendar"
            v-model="focus"
            color="primary"
            :events="events"
            :event-color="getEventColor"
            :type="type"
            @click:event="showEvent"
            @click:more="viewDay"
            @click:date="viewDay"
            @change="updateRange"
            locale="es-ES"
            :weekdays="weekdays"
            :short-weekdays="false"
        ></v-calendar>
        <v-menu
            v-model="selectedOpen"
            :close-on-content-click="false"
            :activator="selectedElement"
            offset-x
        >
          <v-card
              color="grey lighten-4"
              min-width="350px"
              flat
          >
            <v-toolbar
                :color="selectedEvent.color"
                dark
            >
              <v-btn icon @click="DeleteEvent(selectedEvent)">
                <v-icon>mdi-delete</v-icon>
              </v-btn>
              <v-toolbar-title v-html="selectedEvent.name"></v-toolbar-title>
              <v-spacer></v-spacer>
            </v-toolbar>
            <v-card-text>
              <v-form v-if="currentlyEditing !== selectedEvent.id">
                {{selectedEvent.details}}
              </v-form>
              <v-form v-else>
                <v-text-field
                  v-model="selectedEvent.name"
                  label="Edite Nombre"
                  :rules="[rules.required]"
                >
                </v-text-field>
                <textarea-autosize
                    placeholder="Edite Details..."
                    style="width:100%; background-color:white;"
                    :min-height="40"
                    :max-height="100"
                    v-model="selectedEvent.details"
                    :rules="[rules.required]"
                >
                  {{selectedEvent.details}}
                </textarea-autosize>
              </v-form>
            </v-card-text>
            <v-card-actions>
              <v-btn
                  v-if="currentlyEditing !== selectedEvent.id"
                  text
                  color="secondary"
                  @click="selectedOpen = false"
              >
                Cancel
              </v-btn>
              <v-btn
                v-if="currentlyEditing !== selectedEvent.id"
                text
                @click.prevent="editEvent(selectedEvent.id)"
              >
                Editar
              </v-btn>
              <v-btn v-else
                text
                @click="updateEvent(selectedEvent)"
              >
                Guardar Cambios
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-menu>
        <!-- Modal Agregar Evento -->
        <v-dialog v-model="dialog">
          <v-card>
            <v-card-title>Agregar Evento</v-card-title>
            <v-form @submit.prevent="addEvent">
              <v-container>
                  <v-row>
                    <v-col
                        cols="12"
                    >
                      <v-text-field
                          type="text" label="Agregar Nombre" v-model="name"
                          :rules="[rules.required]"
                      >
                      </v-text-field>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col
                        cols="12"
                    >
                      <v-text-field
                          type="text" label="Agregar Detalle" v-model="detail"
                          :rules="[rules.required]"
                      >
                      </v-text-field>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          type="date" label="Inicio del Evento" v-model="start"
                          :rules="[rules.required]"
                      >
                      </v-text-field>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          type="date" label="Fin del Evento" v-model="end"
                          :rules="[rules.required]"
                      >
                      </v-text-field>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          type="color" label="Color del Evento" v-model="color"
                          :rules="[rules.required]"
                      >
                      </v-text-field>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col
                        cols="12"
                    >
                        <v-card-actions>
                          <v-spacer></v-spacer>
                            <v-btn
                                type="submit"
                                dark
                                color="blue darken-1"
                                text
                                @click.stop="dialog = false">Agregar
                            </v-btn>
                        </v-card-actions>
                    </v-col>
                  </v-row>
              </v-container>
              </v-form>

          </v-card>
        </v-dialog>
      </v-sheet>
    </v-col>
  </v-row>
</template>

<script>
import {db} from '../main'

export default {
  name: "Calendario",
  data: () => ({
    focus: '',
    type: 'month',
    typeToLabel: {
      month: 'Mes',
      week: 'Semanal',
      day: 'Día',
      '4day': '4 Días',
    },
    weekdays:[ 1,2,3,4,5,6,0 ],
    selectedEvent: {},
    selectedElement: null,
    selectedOpen: false,
    events: [],
    colors: ['blue', 'indigo', 'deep-purple', 'cyan', 'green', 'orange', 'grey darken-1'],
    names: ['Meeting', 'Holiday', 'PTO', 'Travel', 'Event', 'Birthday', 'Conference', 'Party'],
    name: null,
    detail: null,
    start: null,
    end: null,
    color: '#254786',
    timed: null,
    currentlyEditing: null,
    dialog: false,
    rules: {
      required: value => !!value || 'Required.',
    }
  }),
  mounted () {
    this.$refs.calendar.checkChange()
  },
  created() {
    this.getEvent()
  },
  methods: {
    async updateEvent(elem){
      try {
        await db.collection('events').doc(elem.id).update({
          name: elem.name,
          details: elem.details
        })
        this.selectedOpen = false
        this.currentlyEditing = false
        await this.getEvent()
      }
      catch (e) {
        console.log("Error: " + e)
      }
      finally {

      }
    },
    async editEvent(id){
      this.currentlyEditing=id
    },
    async DeleteEvent(elem){
      try {
        db.collection('events').doc(elem.id).delete()
        this.selectedOpen=false
        this.getEvent()
      }
      catch (e) {

      }
      finally {

      }
    },
    async addEvent(){
      try {
        if(this.name && this.start && this.end){
          await db.collection('events').add({
            name: this.name,
            details: this.detail,
            start: this.start,
            end: this.end,
            color: this.color,
          })
          await this.getEvent()
          this.name=null
          this.details=null
          this.start=null
          this.end=null
          this.color='#2d6ee5'
        }
        else{
          //console.log('no')
        }
      }
      catch (e){
        //console.log('Error ' + e)
      }
      finally {
        //console.log('fin')
      }
    },
    async getEvent(){
      try {
        const snapshot = await db.collection('events').get()
        const events = []
        snapshot.forEach(doc => {
          //console.log(doc.data(), doc.id)
          let eventoData=doc.data()
          eventoData.id=doc.id
          events.push(eventoData)
        })
        this.events = events;
      }
      catch (e){
        //console.log(e)
      }
      finally {

      }
    },
    viewDay ({ date }) {
      this.focus = date
      this.type = 'day'
    },
    getEventColor (event) {
      return event.color
    },
    setToday () {
      this.focus = ''
    },
    prev () {
      this.$refs.calendar.prev()
      this.getEvent()
    },
    next () {
      this.$refs.calendar.next()
      this.getEvent()
    },
    showEvent ({ nativeEvent, event }) {
      const open = () => {
        this.selectedEvent = event
        this.selectedElement = nativeEvent.target
        setTimeout(() => {
          this.selectedOpen = true
        }, 10)
      }

      if (this.selectedOpen) {
        this.selectedOpen = false
        setTimeout(open, 10)
      } else {
        open()
      }

      nativeEvent.stopPropagation()
    },
    updateRange ({ start, end }) {
      const events = []

      const min = new Date(`${start.date}T00:00:00`)
      const max = new Date(`${end.date}T23:59:59`)
      const days = (max.getTime() - min.getTime()) / 86400000
      const eventCount = this.rnd(days, days + 20)

      /*for (let i = 0; i < eventCount; i++) {
        const allDay = this.rnd(0, 3) === 0
        const firstTimestamp = this.rnd(min.getTime(), max.getTime())
        const first = new Date(firstTimestamp - (firstTimestamp % 900000))
        const secondTimestamp = this.rnd(2, allDay ? 288 : 8) * 900000
        const second = new Date(first.getTime() + secondTimestamp)

        events.push({
          name: this.names[this.rnd(0, this.names.length - 1)],
          start: first,
          end: second,
          color: this.colors[this.rnd(0, this.colors.length - 1)],
          timed: !allDay,
        })
      }*/

      this.events = events
    },
    rnd (a, b) {
      return Math.floor((b - a + 1) * Math.random()) + a
    },
  },
}
</script>

