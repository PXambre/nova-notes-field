<template>
    <li class="flex text-left bg-indigo overflow-hidden px-4 py-4">
        <div class="mr-4 flex-shrink-0">
            <img
                class="inline-block h-16 w-16 rounded-full"
                src="https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=facearea&facepad=2&w=256&h=256&q=80"
                alt=""
            />
        </div>
        <div>

            <div class="flex space-x-2">
                <h4 class="text-lg font-semibold">{{ data.author ? data.author.name : '-' }}</h4>
                <icon-eye-off v-if="data.personal"/>

                <!--EDIT-->
                <a class="text-sm font-bold text-yellow-400 cursor-pointer"
                   @click="onEdit"
                >Edit</a>

                <!--DELETE-->
                <a class="text-sm font-bold text-red-500 cursor-pointer"
                   @click="onDelete"
                >Delete</a>
            </div>

            <p
                class="trix-content my-3"
                v-if="data.as_html"
                v-html="data.note"
            ></p>

            <p class="my-3" v-else>{{ data.note }}</p>

            <p class="text-gray-500">{{ createdAt }}</p>

            <div v-if="!isReply" class="w-full my-1">
                <a @click="handleReplyTo" class="no-underline dim text-primary font-bold cursor-pointer">Reply</a>
            </div>

            <div v-if="!isReply" v-for="note in data.notes" class="mt-6 flex">
                <note :data=note :key="note.note" @delete-note="onDelete($event, true)"></note>
            </div>
        </div>

    </li>
</template>

<script>
import moment from 'moment/min/moment-with-locales';
import IconEyeOff from '../icons/IconEyeOff.vue';

export default {
    name: 'note',
    components: {
        IconEyeOff,
    },
    props: {
        data: {
            type: Object,
            required: true,
        },
    },
    data() {
        return {
            isReply: this.data.notable_type === "PDMFC\\NovaNotesField\\Models\\Note"
        }
    },
    created() {
        moment.locale(Nova.config.locale);
    },
    methods: {
        handleReplyTo() {
            this.$emit('reply-to', {
                notable_type: this.data.notable_type,
                notable_id: this.data.notable_id,
                reply_to_id: this.data.id,
                reply_to_name: this.data.author ? this.data.author.name : '-'
            });
        },
        onDelete(data, reply) {

            if(!reply){
                Nova.request().delete('/nova-vendor/notes-field/' + this.data.id).then(response => {
                    this.$toasted.show('Note Removed', {type: 'success'})

                    this.$emit('delete-note', {
                        isReply: this.isReply,
                        notableID: this.data.notable_id,
                        dataID: this.data.id
                    })
                })
            } else {
                this.$emit('delete-note', data)
            }
        },
        onEdit() {
            //get NOTE ID
            //check personal status
            //check html status
            //display note
            //update changes
        }
    },
    computed: {
        createdAt() {
            return `${moment(this.data.created_at).fromNow()} - ${moment(
                this.data.created_at
            ).format('DD/MM/YY HH:mm')}`;
        },
    },
};
</script>

<style scoped src="../../../sass/field.css"></style>
