<template>
    <div id="tasks">
        <v-data-table
            :headers="headers"
            :items="!deleteFlag?tasks:removedTask"
            sort-by="dtcreate"
            sort-desc
            show-select
            v-model="selectedTask"
        >
            <template v-slot:item.done="{ item }">
                <v-btn :disabled="item.delflg === delFlgList.inactive" class="mx-2" fab x-small @click="markAsCompleted(item)">
                    <v-icon v-if="item.status === 'completed'" x-small>mdi-check</v-icon>
                </v-btn>
            </template>
            <template v-slot:item.name="{ item }">
                {{ item.name | longDataFormat }}
            </template>
            <template v-slot:item.desc="{ item }" class="desc">
                {{ item.desc | longDataFormat }}
            </template>
            <template v-slot:item.status="{ item }">
                <template v-if="item.delflg == delFlgList.inactive">
                    <v-chip
                        class="ma-2 wid100"
                        small
                    >
                        <v-icon left small>mdi-delete</v-icon>
                        Deleted
                    </v-chip>
                </template>
                <template v-else>
                    <div v-for="(status, index) in statusList" :key="index">
                        <template v-if="item.status === status.value">
                            <v-chip
                                class="ma-2 wid100"
                                :color="status.color"
                                outlined
                                small
                            >
                                <v-icon left small>{{ status.icon }}</v-icon>
                                {{ status.text }}
                            </v-chip>
                        </template>
                    </div>
                </template>
            </template>
            <template v-slot:top>
                <v-row align="center">
                    <template v-if="!deleteFlag">
                        <v-dialog v-model="markAllDoneDialog" max-width="290">
                            <template v-slot:activator="{ on, attrs }">
                                <v-btn class="ma-2" outlined color="green" v-bind="attrs" v-on="on">
                                    <v-icon left>mdi-check</v-icon>
                                    {{ markAllBtnTitle }}
                                </v-btn>
                            </template>
                            <v-card>
                                <v-card>
                                    <v-card-title class="headline">Confirm</v-card-title>
                                    <v-card-text>
                                        Are you sure you want to mark all these tasks done?
                                    </v-card-text>
                                    <v-card-actions>
                                        <v-spacer></v-spacer>
                                        <v-btn text @click="markAllDoneDialog = false">Cancel</v-btn>
                                        <v-btn color="green darken-1" text @click="markAllDone">Mark all done</v-btn>
                                    </v-card-actions>
                                </v-card>
                            </v-card>
                        </v-dialog>
                        <v-dialog v-model="deleteAllDialog" max-width="290">
                            <template v-slot:activator="{ on, attrs }">
                                <v-btn class="ma-2" outlined color="blue-grey" v-bind="attrs" v-on="on">
                                    <v-icon left>mdi-delete-outline</v-icon>
                                    {{ deleteBtnTitle }}
                                </v-btn>
                            </template>
                            <v-card>
                                <v-card-title class="headline">Confirm</v-card-title>
                                <v-card-text>Are you sure you want to delete all these tasks?</v-card-text>
                                <v-card-actions>
                                    <v-spacer></v-spacer>
                                    <v-btn text @click="deleteAllDialog = false">Cancel</v-btn>
                                    <v-btn color="green darken-1" text @click="deleteAll">Remove</v-btn>
                                </v-card-actions>
                            </v-card>
                        </v-dialog>
                    </template>
                    <template v-else>
                        <v-dialog v-model="recoverAllDialog" max-width="290">
                            <template v-slot:activator="{ on, attrs }">
                                <v-btn class="ma-2" outlined color="blue-grey" v-bind="attrs" v-on="on">
                                    <v-icon left>mdi-restart</v-icon>
                                    {{ recoverBtnTitle }}
                                </v-btn>
                            </template>
                            <v-card>
                                <v-card-title class="headline">Confirm</v-card-title>
                                <v-card-text>Are you sure you want to recover all these tasks?</v-card-text>
                                <v-card-actions>
                                    <v-spacer></v-spacer>
                                    <v-btn text @click="recoverAllDialog = false">Cancel</v-btn>
                                    <v-btn color="green darken-1" text @click="recoverAll">Recover</v-btn>
                                </v-card-actions>
                            </v-card>
                        </v-dialog>
                    </template>
                    <v-spacer></v-spacer>
                    <v-btn
                        color="info"
                        dark
                        class="ma-2"
                        @click="updateTask(defaultTask)"
                    >
                        <v-icon left>mdi-playlist-plus</v-icon>
                        {{ newTaskLabel }}
                    </v-btn>
                </v-row>
                <v-switch
                    v-model="deleteFlag"
                    label="Show deleted tasks"
                    color="info"
                    hide-details
                    class="showDeleted"
                ></v-switch>
                <v-form ref="taskForm">
                    <v-dialog v-model="editDialog" max-width="500">
                        <v-card>
                            <v-card-title>
                                <span class="headline">{{ formTitle }}</span>
                            </v-card-title>
                            <v-card-text>
                                <v-container>
                                    <v-row>
                                        <v-col cols="12">
                                            <v-text-field v-model="editedTask.name" required
                                                          :rules="[v => !!v || 'Please enter the task name.']"
                                                          :disabled="editedTask.delflg === delFlgList.inactive">
                                                <template v-slot:label>
                                                    <span>Task Name <span class="red--text">*</span></span>
                                                </template>
                                            </v-text-field>
                                        </v-col>
                                        <v-col cols="12">
                                            <v-textarea v-model="editedTask.desc" required
                                                        :rules="[v => !!v || 'Please enter the task description.']"
                                                        :disabled="editedTask.delflg === delFlgList.inactive">
                                                <template v-slot:label>
                                                    <span>Task Description <span class="red--text">*</span></span>
                                                </template>
                                            </v-textarea>
                                        </v-col>
                                        <v-col cols="12">
                                            <v-select
                                                v-model="editedTask.status"
                                                :items="statusList"
                                                :rules="[v => !!v || 'Please select a status.']"
                                                :disabled="editedTask.delflg === delFlgList.inactive"
                                                required
                                                item-value="value"
                                                item-text="text"
                                            >
                                                <template v-slot:label>
                                                    <span>Task Status <span class="red--text">*</span></span>
                                                </template>
                                            </v-select>
                                        </v-col>
                                    </v-row>
                                </v-container>
                                <small><span class="red--text">*</span> indicates required field</small>
                            </v-card-text>

                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn text @click="cancel('edit')">Cancel</v-btn>
                                <v-btn color="green" v-if="editedTask.delflg === delFlgList.active && editedId > -1"
                                       text @click="removeTask(editedTask)">Remove
                                </v-btn>
                                <v-btn color="blue" v-if="editedTask.delflg === delFlgList.active" text
                                       @click="save">Save
                                </v-btn>
                                <v-btn color="blue" v-if="editedTask.delflg === delFlgList.inactive" text
                                       @click="save">Recover
                                </v-btn>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                </v-form>
                <v-dialog
                    v-model="deleteDialog"
                    max-width="350"
                >
                    <v-card>
                        <v-card-title class="headline">{{ editedTask.name }}
                        </v-card-title>
                        <v-card-text>
                            Are you sure you want to delete this task?
                        </v-card-text>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn text @click="cancel('delete')">Cancel</v-btn>
                            <v-btn color="green darken-1" text @click="remove">Remove</v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
            </template>
            <template v-slot:item.actions="{ item }">
                <v-icon @click="updateTask(item)">mdi-file-document-edit-outline</v-icon>
                <v-icon v-if="item.delflg === delFlgList.active" @click="removeTask(item)">mdi-delete-outline</v-icon>
            </template>
        </v-data-table>
    </div>
</template>
<script>

    export default {
        name: 'Tasks',
        data() {
            return {
                deleteFlag       : false,
                tasks            : [],
                removedTask      : [],
                tempTask         : [],
                tempRemovedTask  : [],
                selectedTask     : [],
                newTaskLabel     : 'New Task',
                editTaskLabel    : 'Edit Task',
                statusList       : [
                    {text: 'New', value: 'new', color: 'red', icon: 'mdi-alert-decagram',},
                    {text: 'In Progress', value: 'pending', color: 'indigo darken-3', icon: 'mdi-progress-clock'},
                    {text: 'Completed', value: 'completed', color: 'success', icon: 'mdi-checkbox-marked-circle'},
                ],
                editDialog       : false,
                deleteDialog     : false,
                markAllDoneDialog: false,
                deleteAllDialog  : false,
                recoverAllDialog  : false,
                headers          : [
                    {text: 'Mark as done', value: 'done', sortable: false, width: '105'},
                    {text: 'Task Name', value: 'name', width: '200'},
                    {text: 'Task Description', value: 'desc', width: '200'},
                    {text: 'Date Created', value: 'dtcreate', width: '200'},
                    {text: 'Status', value: 'status', width: '150'},
                    {text: 'Actions', value: 'actions', sortable: false, width: '100'},
                ],
                editedId         : -1,
                defaultIndex     : -1,
                editedTask       : {
                    id      : 0,
                    name    : '',
                    desc    : '',
                    status  : 'new',
                    delflg  : 0,
                    dtcreate: '0000-00-00 00:00:00',
                    dtupdate: '0000-00-00 00:00:00'
                },
                defaultTask      : {
                    id      : 0,
                    name    : '',
                    desc    : '',
                    status  : 'new',
                    delflg  : 0,
                    dtcreate: '0000-00-00 00:00:00',
                    dtupdate: '0000-00-00 00:00:00'
                },
                delFlgList       : {'active': 0, 'inactive': 1},
            }
        },

        computed: {
            formTitle() {
                return this.editedId > -1 ? this.editTaskLabel : this.newTaskLabel
            },

            markAllBtnTitle() {
                return this.selectedTask.length > 0?'Mark as done':'Mark all as done'
            },

            deleteBtnTitle() {
                return this.selectedTask.length > 0?'Delete':'Delete All'
            },

            recoverBtnTitle() {
                return this.selectedTask.length > 0?'Recover':'Recover All'
            },
        },

        watch: {
            editDialog(val) {
                val || this.cancel()
            },

            deleteDialog(val) {
                val || this.cancel()
            },

            tasks: {
                handler: function () {
                    localStorage.tasks = JSON.stringify(this.tasks)
                },
                deep   : true
            },

            removedTask: {
                handler: function () {
                    localStorage.removedTask = JSON.stringify(this.removedTask)
                },
                deep   : true
            }
        },

        filters: {
            longDataFormat: (val) => {
                return val && val.length > 101 ? val.substr(0, 100)+'...' : val
            },
        },

        created() {
            if (localStorage.tasks) {
                this.tasks = JSON.parse(localStorage.tasks)
            } else {
                let date = this.getDateTime(new Date)
                for (let i = 1; i <= 1000; i++) {
                    this.tasks.push({
                        id      : i,
                        name    : 'Task Name ' + i,
                        desc    : 'Task Description ' + i,
                        status  : i % 2 == 0 ? 'completed' : 'pending',
                        delflg  : 0,
                        dtcreate: date,
                        dtupdate: '0000-00-00 00:00:00',
                    })
                }

                localStorage.tasks = JSON.stringify(this.tasks)
            }

            if (localStorage.removedTask) {
                this.removedTask = JSON.parse(localStorage.removedTask)
            }
        },

        methods: {
            /**
             * To mark all tasks done
             */
            markAllDone() {
                this.tasks.map((item) => {
                    let selectedTask = this.selectedTask
                    var selected = selectedTask.find(x => x.id === item.id)
                    if (selectedTask.length < 1 || (selected && selected.id === item.id)) {
                        item.status = 'completed'
                    }
                    return item
                });
                this.selectedTask = []
                this.markAllDoneDialog = false
            },

            /**
             * To delete all tasks that are
             * selected or just delete everything
             * if no task is selected
             */
            deleteAll() {
                this.tasks = this.tasks.map((item) => {
                    let selectedTask = this.selectedTask
                    var selected = selectedTask.find(x => x.id === item.id)
                    if (selectedTask.length < 1 || (selected && selected.id === item.id)) {
                        item.delflg = this.delFlgList.inactive
                        this.removedTask.push(item)
                    }
                    return item
                }).filter(item => item.delflg < 1)
                this.selectedTask = []
                this.deleteAllDialog = false
            },

            /**
             * To recover all tasks that are
             * selected or just recover everything
             * if no task is selected
             */
            recoverAll() {
                if (this.selectedTask.length < 1) {
                    this.removedTask.map((item) => {
                        item.delflg = this.delFlgList.active
                        this.tasks.push(item)
                    })
                    this.removedTask = []
                } else {
                    this.selectedTask.map((item) => {
                        item.delflg = this.delFlgList.active
                        this.tasks.push(item)
                    });
                    this.removedTask = this.removedTask.filter((item) => {
                        var selected = this.selectedTask.find(x => x.id === item.id)
                        return typeof selected === 'undefined'
                    })
                    this.selectedTask = []
                }
                this.recoverAllDialog = false
            },

            /**
             * To mark completed task
             * from the table itself
             */
            markAsCompleted(item) {
                this.tasks = this.tasks.map((val) => {
                    if (val.id == item.id) {
                        item.status = item.status !== 'completed' ? 'completed' : 'pending'
                        item.dtupdate = this.getDateTime(new Date())
                        return item
                    } else {
                        return val
                    }
                });
            },

            /**
             * To update the data of
             * the task which will
             * open the update dialog
             */
            updateTask(item) {
                this.editedId = item.id == 0 ? -1 : item.id
                this.editedTask = Object.assign({}, item)
                this.editDialog = true
            },

            /**
             * To remove the task
             * which will open the
             * delete dialog
             */
            removeTask(item) {
                this.editedId = item.id
                this.editedTask = Object.assign({}, item)
                this.deleteDialog = true
            },

            /**
             * If cancel button is
             * clicked inside Edit and
             * Delete dialog
             */
            cancel(type) {
                this.editedId = this.defaultIndex
                if (type === 'edit') this.editDialog = false
                this.deleteDialog = false
            },

            /**
             * If remove button is clicked
             * inside the delete dialog
             */
            remove() {
                this.tasks = this.tasks.map((item) => {
                    if (item.id == this.editedId) {
                        item.delflg = this.delFlgList.inactive
                        item.dtupdate = this.getDateTime(new Date())
                        this.removedTask.push(item)
                    }
                    return item
                }).filter(item => item.delflg < 1)
                this.editedId = this.defaultIndex
                this.deleteDialog = false
                this.editDialog = false
            },

            /**
             * If save button is clicked inside
             * the edit or new dialog
             */
            save() {
                // for validation
                if (this.$refs.taskForm.validate()) {
                    if (this.editedId > -1) {
                        this.editedTask.dtupdate = this.getDateTime(new Date())
                        if (this.editedTask.delflg === this.delFlgList.inactive) {
                            this.editedTask.delflg = this.delFlgList.active
                            this.tasks.push(this.editedTask)
                            this.tasks.sort((a, b) => {
                                let dateA = new Date(a.dtcreate),
                                    dateB = new Date(b.dtcreate)
                                return dateB - dateA || a.id - b.id
                            })
                            this.removedTask = this.removedTask.filter(item => this.editedTask.id !== item.id)
                        }

                        this.tasks = this.tasks.map(item => {
                            return item.id === this.editedTask.id ? this.editedTask : item
                        })
                    } else {
                        let taskMaxId = this.tasks.reduce((maxId, item) => Math.max(maxId, item.id), 0) + 1
                        let removedMaxId = this.removedTask.reduce((maxId, item) => Math.max(maxId, item.id), 0)
                        let maxId = taskMaxId === removedMaxId ? taskMaxId + 1 : taskMaxId
                        if (this.tasks.length < 1) {
                            maxId = removedMaxId + 1
                        }
                        this.editedTask.id = maxId
                        this.editedTask.dtcreate = this.getDateTime(new Date())
                        this.tasks.unshift(this.editedTask)
                    }

                    this.editedId = this.defaultIndex
                    this.deleteFlag = false
                    this.editDialog = false
                }
            },

            /**
             * To get the date which
             * will return a format of
             * 'YYYY-mm-dd hh:ii:ss'
             *
             * @param   object date
             * @returns string
             */
            getDateTime(date) {
                let format = (val) => val < 10 ? '0' + val : val
                let yy = date.getFullYear()
                let mm = format(date.getMonth() + 1)
                let dd = format(date.getDate())

                let hh = format(date.getHours())
                let ii = format(date.getMinutes())
                let ss = format(date.getSeconds())

                return yy + "-" + mm + "-" + dd + " " + hh + ":" + ii + ":" + ss
            },
        },
    }
</script>

<style scoped>
    #tasks {
        margin: 15px 10px;
    }
    .wid100 {
        width: 100px;
    }

    .desc {
        white-space: pre
    }

    .showDeleted {
        margin-bottom: 16px;
    }
</style>
