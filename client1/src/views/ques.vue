<template>
<div>
    <div class="text-center blue lighten-5">
        <h1>Knowledge test</h1>
    </div>
    <v-row>
        <v-col cols="9">
            <v-container v-for="(item, i) in ques" :key="i">
                <v-card class="mx-auto" outlined>
                    <v-list-item three-line>
                        <v-list-item-content>
                            <v-list-item-title>
                                <v-row>
                                    <v-col cols="5">
                                        {{ i + 1 }}.) {{ item.question }} ({{ item.id }})
                                    </v-col>
                                    <v-col cols="5">
                                        <div v-if="item.selected.length > 0">
                                            <span v-if="!item.result"> คุณตอบผิด </span>
                                            <span v-else> คุณตอบถูก </span>
                                        </div>
                                    </v-col>
                                    <v-spacer />
                                    <v-col cols="2" class="grey--text mb-2 text-right">Select ({{ item.ans.length }})</v-col>
                                </v-row>
                            </v-list-item-title>
                            <v-list-item-subtitle>
                                <v-container>
                                    <v-row>
                                        <v-col v-for="(xitem, i) in item.choices" :key="i" cols="6">
                                            <v-checkbox v-if="showans" v-model="item.ans" readonly color="success" :label="xitem.name" :value="xitem.id" />
                                            <v-checkbox v-else v-model="item.selected" color="success" :label="xitem.name + '(' + xitem.id + ')'" :value="xitem.id" @change="checkans(item)" />
                                        </v-col>
                                    </v-row>
                                </v-container>
                            </v-list-item-subtitle>
                        </v-list-item-content>
                    </v-list-item>
                </v-card>
            </v-container>
        </v-col>
        <v-col cols="3">
            <v-container>
                <v-card class="mx-auto" outlined>
                    <v-list-item three-line>
                        <v-list-item-content>
                            <div class="overline mb-1"></div>
                            <v-list-item-title class="headline mb-2"> </v-list-item-title>
                            <v-list-item-subtitle> </v-list-item-subtitle>
                        </v-list-item-content>
                    </v-list-item>

                    <v-card-actions>
                        <v-spacer />
                        <v-switch v-model="showans" label="เฉลย"></v-switch>
                    </v-card-actions>
                </v-card>
            </v-container>
        </v-col>
    </v-row>
</div>
</template>

<script>
const axios = require("axios");
import {
    msalMixin
} from "vue-msal";
export default {
    name: "question",
    mixins: [msalMixin],
    data: () => ({
        showans: false,
        ques: [],
    }),
    components: {},
    methods: {
        random(item) {
            // let id = 0;
            while (item.length) {
                let rendomItem = (Math.random() * item.length) | 0;
                let newItem = item[rendomItem].choices;

                let newchoices = [];
                while (newItem.length) {
                    let randomChoices = (Math.random() * newItem.length) | 0;
                    newchoices.push(newItem[randomChoices]);
                    newItem.splice(randomChoices, 1);
                }
                item[rendomItem].choices = newchoices;
                this.ques.push(item[rendomItem]);
                item.splice(rendomItem, 1);
            }
        },
        async getQuest() {
            const quest = await axios.get("http://172.18.1.75:3000/question");
            // console.log(quest);
            // this.ques = quest.data;
            this.random(quest.data);
        },
        checkans(item) {
            let xans = item.ans.sort().toString();
            let xselect = item.selected.sort().toString();
            item.result = xans == xselect;
        },
        async checkauthen() {
            if (this.$msal.isAuthenticated()) {
                this.getQuest();
            } else {
                this.$msal.signIn();
            }
        },
    },
    mounted() {
        this.checkauthen();
    },
};
</script>
