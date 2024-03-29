<template>
    <x-layouts-panel v-if="ticket && ticket.user">
        <template v-slot:asideLeft>
            <x-section-header>
                <template v-slot:title>
                    Details
                </template>
            </x-section-header>

            <x-table>
                <thead>
                    <x-table-row>
                        <x-table-header>Ticket Details</x-table-header>
                    </x-table-row>
                </thead>

                <tbody>
                    <x-table-row>
                        <x-table-data>
                            <ul class="space-y-4">
                                <li>
                                    <div class="grid grid-cols-2 gap-2">
                                        <div>
                                            Status
                                            <select class="max-w-lg block focus:ring-blue-500 focus:border-blue-500 w-full shadow-sm sm:max-w-xs sm:text-sm border-gray-300 rounded-md" v-model="ticket.status_id">
                                                <option :value="status.id" v-for="(status, statusIndex) in statuses" :key="'status_' + statusIndex">
                                                    {{ status.name }}
                                                </option>
                                            </select>
                                        </div>

                                        <div>
                                            Priority
                                            <select class="max-w-lg block focus:ring-blue-500 focus:border-blue-500 w-full shadow-sm sm:max-w-xs sm:text-sm border-gray-300 rounded-md" v-model="ticket.priority_id">
                                                <option :value="priority.id" v-for="(priority, priorityIndex) in priorities" :key="'priority_' + priorityIndex">
                                                    {{ priority.name }}
                                                </option>
                                            </select>
                                        </div>
                                    </div>
                                </li>

                                <li>
                                    Department
                                    <select class="max-w-lg block focus:ring-blue-500 focus:border-blue-500 w-full shadow-sm sm:max-w-xs sm:text-sm border-gray-300 rounded-md" v-model="ticket.department_id">
                                        <option :value="department.id" v-for="(department, departmentIndex) in departments" :key="'department_' + departmentIndex">
                                            {{ department.name }}
                                        </option>
                                    </select>
                                </li>
                            </ul>
                        </x-table-data>
                    </x-table-row>
                </tbody>
            </x-table>
        </template>

        <template v-slot:default>
            <x-section-header>
                <template v-slot:title>
                    #{{ ticket.id }} - {{ ticket.subject }} 
                    <x-link :to="{ name: 'users.edit', params: { user: ticket.user.id } }" v-if="ticket.user">
                        ({{ ticket.user.name }})
                    </x-link>
                </template>
            </x-section-header>

            <x-section-toolbar>
                <x-button color="white">
                    Close Ticket
                </x-button>
            </x-section-toolbar>

            <div class="h-full border-b border-gray-200 overflow-x-hidden overflow-y-auto">
                <ul class="h-full py-4 space-y-2 sm:px-6 sm:space-y-4 overflow-x-hidden overflow-y-auto" v-if="ticket.messages">
                    <li class="px-4 py-6 shadow sm:rounded-lg sm:px-6 max-w-3xl " :class="(message.message_type_id == 1 ? 'bg-white':'bg-yellow-100') + ' ' + (message.user.is_customer ? 'mr-auto':'ml-auto')" v-for="(message, messageIndex) in ticket.messages" :key="'message_' + messageIndex">
                        <div class="sm:flex sm:justify-between sm:items-baseline">
                            <h3 class="text-base font-medium">
                                <span class="text-gray-900">{{ message.user.name }}</span>
                            </h3>

                            <p class="mt-1 text-sm text-gray-600 whitespace-nowrap sm:mt-0 sm:ml-3">
                                <time datetime="2021-01-28T19:24">{{ message.source_created_at }}</time>
                            </p>
                        </div>

                        <div class="mt-4 space-y-6 text-sm text-gray-800" v-if="message">
                            <x-iframe :content="message.content"/>
                        </div>
                    </li>
                </ul>
            </div>

            <div class="py-4 bg-white">
                <x-container>
                    <div class="flex flex-col space-y-2">
                        <textarea rows="5" class="block w-full py-2 px-4 border border-gray-300 rounded-md leading-5 bg-white shadow-sm placeholder-gray-500 focus:outline-none focus:placeholder-gray-400 focus:ring-1 focus:ring-blue-600 focus:border-blue-600 sm:text-sm" v-model="messageForm.content"></textarea>
                        <div class="flex items-center justify-between flex-shrink-0">
                            <x-text color="muted">
                                <span class="text-sm italic">
                                    &check; Draft Saved at 2:00PM
                                </span>
                            </x-text>

                            <div class="space-x-2">
                                <x-button color="white" @click.prevent="submitMessageForm(messageForm, 2)">
                                    Send Note
                                </x-button>

                                <x-button color="primary" @click.prevent="submitMessageForm(messageForm, 1)">
                                    Send Reply
                                </x-button>
                            </div>
                        </div>
                    </div>
                </x-container>
            </div>
        </template>
    </x-layouts-panel>
</template>

<script>
import { mapGetters } from "vuex";

export default {
    data() {
        return {
            ticket: {
                department_id: null,
                statis_id: null,
                priority_id: null,
            },
            messageForm: {
                content: "",
                channel_id: null,
                ticket_id: null,
                message_type_id: 1,
                user_id: 1, // Replace this with logged in user later on.
            },
        };
    },
    computed: {
        ...mapGetters("statusModule", {
            statuses: "getItems",
        }),
        ...mapGetters("priorityModule", {
            priorities: "getItems",
        }),
        ...mapGetters("departmentModule", {
            departments: "getItems",
        }),
    },
    created() {
        this.$store.dispatch("departmentModule/fetchAllItems");
        this.$store.dispatch("priorityModule/fetchAllItems");
        this.$store.dispatch("statusModule/fetchAllItems");
        this.$store.dispatch("ticketModule/fetchOneItem", this.$route.params.ticket)
            .then((ticket) => {
                // Populate ticket information.
                this.ticket = ticket;
                // Populate message information.
                this.messageForm.ticket_id = ticket.id;
                this.messageForm.channel_id = ticket.channel_id;
            })
            .catch((error) => {
                console.log(error);
            });
    },
    methods: {
        submitMessageForm(payload, message_type_id) {
            payload = {
                ...payload,
                ...{ message_type_id },
            };

            this.$store.dispatch("messageModule/storeItem", payload)
                .then(() => {
                    // ...
                })
                .catch(() => {
                    // ...
                })
                .finally(() => {
                    // ...
                });
        },
    },
};
</script>