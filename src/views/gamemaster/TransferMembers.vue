<template>
    <div class="container" v-if="needSelection">
        <member-select @member-selected="select" filter="all"></member-select>
    </div>
    <div class="container" v-else>
        <div class="row" v-for="user in users" :key="user.id">
            {{user.id}}: {{user.get('username')}}: {{user.get('email')}}
            <button v-on:click="pickUser(user)">Pick {{user.id}}</button>
        </div>
    </div>
</template>

<script>
    import Parse from 'parse';
    let Member = Parse.Object.extend("Member");
    import MemberSelect from '@/components/MemberSelect.vue'

    export default {
        name: "GameMasterTransferMembers",
        components: { MemberSelect },
        data: function() {
            return {
                members: [],
                memberIds: [],
                memberId: "",
                users: []
            }
        },
        computed: {
            needSelection() {
                return this.memberId === "";
            }
        },
        mounted: async function() {
            await this.fetch();
        },
        methods: {
            async fetch() {
                const q = new Parse.Query(Member).select("id");
                let memberIds = []
                await q.each((member) => {
                    memberIds.push(member.id);
                });
                this.members = [];
                for (let memberId of memberIds) {
                    let {member} = await this.$store.dispatch('loadOrUseMember', {
                        memberId: memberId
                    });
                    this.members = [member].concat(this.members);
                    this.memberIds = [member.id].concat(this.memberIds);
                }
            },
            async select(memberId) {
                const q = new Parse.Query(Parse.User);
                this.users = [];
                await q.each((user) => {
                    this.users = [user].concat(this.users);
                });
                this.memberId = memberId;
            },
            async pickUser(user) {
                await Parse.Cloud.run('giveAllMembersGamemasterAccess');
                await Parse.Cloud.run('transferMemberTo', {
                    userId: user.id,
                    memberId: this.memberId
                });
            }
        }
    }
</script>

<style scoped>

</style>