<template>
    <div class="content pl-4">
        <div class="card">
            <div class="card-body p-1">
                <h5 class="card-title">Meeting Minutes</h5>
                <textarea class="my-3 note-details" name="noteDetails" v-model="noteDetails" rows="5"
                          placeholder="Insert your note"></textarea>
                <!-- <vue-editor v-model="noteDetails"></vue-editor> -->
                <button type="button" class="btn btn-outline-secondary submit-note-details"
                        @click="submitNotes()">Submit
                </button>
            </div>
        </div>
        <div class="row">
            <div class="col-3">
                <P class="my-3 text-center">This Week Meeting Minutes</P>
                <template v-for="note in notes" >
                    <div v-if="isThisWeekTask(note, '1')" class="card my-2" :key="note.id">
                        <div class="card-body note-list-item p1">
                            <p class="card-title">{{getDateFormate(note.submitTime)}}
                                <span class="float-right" style="cursor: pointer" @click="removeNote(note.id)">X</span>
                            </p>
                            <div>{{note.note}}</div>
                        </div>
                    </div>
                </template>
            </div>
            <div class="col-3">
                <P class="my-3 text-center"></P>
                <template v-for="note in notes" >
                    <div v-if="isThisWeekTask(note, '2')" class="card my-2" :key="note.id">
                        <div class="card-body note-list-item p1">
                            <p class="card-title">{{getDateFormate(note.submitTime)}}
                                <span class="float-right" style="cursor: pointer" @click="removeNote(note.id)">X</span>
                            </p>
                            <div>{{note.note}}</div>
                        </div>
                    </div>
                </template>
            </div>
            <div class="col-3">
                <!-- <P class="my-3 text-center">Last Week Notes</P> -->
                <template v-for="note in notes" >
                    <div v-if="isThisWeekTask(note, '3')" class="card my-2" :key="note.id">
                        <div class="card-body note-list-item p1">
                            <p class="card-title">{{getDateFormate(note.submitTime)}}
                                <span class="float-right" style="cursor: pointer" @click="removeNote(note.id)">X</span>
                            </p>
                            <div>{{note.note}}</div>
                        </div>
                    </div>
                </template>
            </div>
            <div class="col-3">
                <template v-for="note in notes" >
                    <div v-if="isThisWeekTask(note, '4')" class="card my-2" :key="note.id">
                        <div class="card-body note-list-item p1">
                            <p class="card-title">{{getDateFormate(note.submitTime)}}
                                <span class="float-right" style="cursor: pointer" @click="removeNote(note.id)">X</span>
                            </p>
                            <div>{{note.note}}</div>
                        </div>
                    </div>
                </template>
            </div>
        </div>
    </div>
</template>

<script>
    import { mapActions, mapGetters } from "vuex";
    import moment from "moment";
    import { VueEditor } from "vue2-editor";

    export default {
        components: {

            VueEditor
        },
        data() {
            return {
                noteDetails: "",
                getDateFormate(date) {
                    if (date) {
                        return moment(date).format('MM/DD/YYYY h:mm:SS A');
                    }
                },
                isThisWeekTask(note, number) {
                    let todayweekDay = moment().isoWeekday();
                    let startdate = moment().subtract(todayweekDay, "days").toISOString();
                    let today = moment().toISOString();
                    let secondStartDate = moment(startdate).subtract(7, "days").toISOString(),
                        thirdStartDate = moment(secondStartDate).subtract(7, "days").toISOString(),
                        fourStartDate = moment(thirdStartDate).subtract(7, "days").toISOString();

                    if (number === '1') {
                        if (moment(note.submitTime) > moment(startdate) && moment(today) > moment(note.submitTime)) {
                            return true;
                        } else return false;
                    } else if( number === '2') {
                        if (moment(note.submitTime) > moment(secondStartDate) && moment(startdate) > moment(note.submitTime)) {
                            return true;
                        } else return false;
                    } else if( number === '3') {
                        if (moment(note.submitTime) > moment(thirdStartDate) && moment(secondStartDate) > moment(note.submitTime)) {
                            return true;
                        } else return false;
                    } else if( number === '4') {
                        if (moment(note.submitTime) > moment(fourStartDate) && moment(thirdStartDate) > moment(note.submitTime)) {
                            return true;
                        } else return false;
                    }
                }
            };
        },
        computed: {
            ...mapGetters("notes", ["notes"]),
            notes() {
                return this.$store.state.notes.notes;
            }
        },
        methods: {
            ...mapActions("notes",["FETCH_NOTES", "ADD_NOTES", "DELETE_NOTES"]),
            async submitNotes() {
                var date = new Date();

                let data = {
                    note: this.noteDetails,
                    submitTime: date.toISOString(),
                    projectId: this.projectId
                }

                if (data.note) {
                    await this.ADD_NOTES(data).then(res => {
                        console.log("res", res);
                        this.FETCH_NOTES({projectId: this.projectId});
                        this.noteDetails = "";
                    })
                        .catch(err => {
                            console.log(`add : ${err}`);
                        });
                }
            },
            async removeNote(noteId) {
                if (noteId) {
                    var isOkPress = confirm("Are you sure you want to delete this note? You will not get back");

                    if (isOkPress === true) {
                        this.DELETE_NOTES({id: noteId}).then(res => {
                            console.log(res);
                            this.FETCH_NOTES({projectId: this.projectId});
                        })
                    }
                }
            },
        },
        created() {
            let obj = JSON.parse(localStorage.getItem('vuex'));
            this.projectId = obj && obj.project && obj.project.selectedProject && obj.project.selectedProject.id ? obj.project.selectedProject.id : "";

            if (!this.projectId) this.$router.replace(`/home`)
            this.FETCH_NOTES({projectId: this.projectId});
        }
    };
</script>

<style lang="scss" scoped>

    .note-details {
        width: 100%;
        border: 1px solid #ccc;
        border-radius: 5px;
        padding: 10px;
    }
    .submit-note-details {
        width: 100%;
    }

    .note-list-item {
        border: 1px solid #ccc;
        border-radius: 5px;
    }
</style>
