<template>
    <div class="container mt-5">
        <div class="row justify-content-center">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">Calendar</div>
                    <div class="card-body">
                        <h4 class="text-center">
                            <button @click="previousClicked(firstDay)" class="btn btn-default btn-lg"><-Previous</button>
                            -
                            <button @click="nextClicked(lastDay)" class="btn btn-default btn-lg"> Next-></button>

                        </h4>
                        <table class="table table-bordered">
                            <thead>
                            <tr>
                                <th class="calendar-cell" v-for="day in weekDays" :key="day">
                                    {{ day }}
                                </th>
                            </tr>
                            </thead>
                            <tbody>
                                <tr>
                                    <td @click="eventClicked(day)" class="calendar-cell" v-for="day in weekDays" :key="day">
                                        <span v-for="event in eventListings" :key="event.id">
                                            <div class="alert alert-primary" role="alert" v-if="day === getMDYFormat(event.event_date)">
                                                {{ event.note }}
                                            </div>
                                        </span>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <div id="myModal" :class="modalClass">

            <!-- Modal content -->
            <div class="content">
                <span @click="closeModal()" class="close">&times;</span>
                <h4>Add Event</h4>
                <form action=""  @submit.prevent="submitEvent">
                    <p>Date Selected: {{ dateSelected }}</p>

                    <textarea v-model="event.note" required class="form-control" rows="2" name="note" placeholder="Enter Note"></textarea>

                    <button class="btn btn-primary mt-2" type="submit">Submit</button>
                </form>

            </div>

        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            activeDay:'',
            firstDay:'',
            lastDay:'',
            weekDays:[],
            previousActive:false,
            modalClass:'custom-modal hidden',
            dateSelected: '',
            event: {
                event_date:'',
                note:''
            },
            eventListings: []
        }
    },
    mounted() {
        this.getEvents();
        this.firstDay = this.$moment().startOf('week').format('D.M.Y');
        this.getWeek(this.firstDay);
    },
    methods: {
        getWeek(activeDay) {
            console.log(activeDay+' active day');
            this.weekDays = [];
            if (this.previousActive) {
                for (let i = 7; i > 0; i--) {
                    let new_date = '';
                    new_date = this.$moment(activeDay, "D.M.Y").subtract(i, 'days').format('D.M.Y');


                    this.weekDays.push(new_date);

                    if (i === 7) {
                        this.firstDay = new_date;
                    }

                    if (i === 1) {
                        this.lastDay = new_date;
                    }
                }
            } else {
                for (let i = 1; i <= 7; i++) {

                    let new_date ='';
                    new_date = this.$moment(activeDay, "D.M.Y").add(i, 'days').
                    format('D.M.Y');


                    this.weekDays.push(new_date);

                    if (i === 1) {
                        this.firstDay = new_date;
                    }

                    if (i === 7) {
                        this.lastDay = new_date;
                    }
                }
            }

            console.log(this.weekDays);

        },
        eventClicked(day) {
            this.modalClass = 'custom-modal';
            this.dateSelected = day;
            this.event.event_date = this.$moment(day, "D.M.Y").format('Y-M-D');

        },
        nextClicked(activeDay) {
            this.previousActive = false;
            this.getWeek(activeDay);
        },
        previousClicked(activeDay) {
            this.previousActive = true;
            this.getWeek(activeDay);
        },
        closeModal() {
            this.modalClass = 'custom-modal hidden';
        },
        submitEvent() {
            this.$http.post('/api/events', this.event).then(res => {
                if (res.data.success) {
                    this.modalClass = 'custom-modal hidden';
                    this.clearModal();
                    this.getEvents();
                }
            }, err => {
                console.log(err);
            });
        },
        clearModal() {
            this.event = {
                event_date:'',
                note:''
            }
        },
        getEvents() {
            this.$http.get('/api/events', this.event).then(res => {
                this.eventListings = res.data.data;
                console.log(this.eventListings);
            }, err => {
                console.log(err);
            });
        },
        getMDYFormat(day) {
            return this.$moment(day, "Y-M-D").format('D.M.Y');
        }

    }
}
</script>

<style>
    td {
        height: 400px;
    }

    .hidden {
        display:none;
    }
    .custom-modal {
        position: fixed;
        z-index: 1;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        overflow: auto;
        background-color: rgb(0,0,0);
        background-color: rgba(0,0,0,0.4);
    }

    .custom-modal .content {
        background-color: #fefefe;
        margin: 15% auto;
        padding: 20px;
        border: 1px solid #888;
        width: 50%;
    }

    .custom-modal .close {
        color: #aaa;
        float: right;
        font-size: 28px;
        font-weight: bold;
    }

    .custom-modal .close:hover,
    .custom-modal .close:focus {
        color: black;
        text-decoration: none;
        cursor: pointer;
    }
</style>
