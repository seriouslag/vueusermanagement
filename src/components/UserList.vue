<template>
  <div>
    <h1>Users</h1>
    <div v-if="user">
      <p>Modifiy users for your tenant on this page.</p>
      <p>
        <v-btn
          color="primary"
          :to="{name: 'userNew'}"
        >
          Add New User
        </v-btn>
        <v-btn
          color="secondary"
          @click="fetchUsers()"
        >
          Refresh
        </v-btn>
      </p>
      <div v-if="users.length > 0">
        <v-data-table
          :headers="headers"
          :items="users"
        >
          <template
            slot="items"
            slot-scope="props"
          >
            <td>{{ props.item.id }}</td>
            <td>{{ props.item.firstName }}</td>
            <td>{{ props.item.lastName }}</td>
            <td>{{ props.item.email }}</td>
            <td>
              <div
                v-for="authority in props.item.authorities"
                :key="authority.authority"
              >
                {{ authority.authority }}
              </div>
            </td>
            <td>
              <v-btn
                color="success"
                tag="a"
                :to="{ name: 'userEdit', params: { id: props.item.id }}"
              >
                Edit
              </v-btn>
              <v-btn
                color="error"
                href=""
                @click="deleteUser(props.item.id)"
              >
                Delete
              </v-btn>
            </td>
          </template>
        </v-data-table>
      </div>
      <div v-else>
        <p>No users.</p>
      </div>
    </div>
    <div v-else>
      Please log in to edit your users.
    </div>
    <yes-no-dialog
      v-model="showDialog"
      :text="`Remove ${selectedUser.firstName} ${selectedUser.lastName}`"
      @dialogResponse="handleDialogResponse"
    />
  </div>
</template>

<script>
import YesNoDialog from '@/components/YesNoDialog.vue'
const UserList = {
  components: {
    YesNoDialog
  },
  metaInfo: {
    title: 'User List'
  },
  data () {
    return {
      headers: [
        { text: 'ID', sortable: true, value: 'id' },
        { text: 'First Name', sortable: true, value: 'firstName' },
        { text: 'Last Name', sortable: true, value: 'lastName' },
        { text: 'Email Address', sortable: true, value: 'email' },
        { text: 'Role', sortable: true, value: 'authorities' },
        { text: 'Actions', sortable: false }],
      showDialog: false,
      selectedUser: {}
    }
  },
  computed: {
    user: function () {
      return this.$store.getters.user
    },
    users: function () {
      return this.$store.getters['User/users']
    }
  },
  created () {
  },
  methods: {
    async deleteUser (id) {
      this.selectedUser = await this.$store.dispatch('User/getUser', id)
      this.showDialog = true
    },
    async fetchUsers () {
      await this.$store.dispatch('User/getUsers')
    },
    async handleDialogResponse (response) {
      this.showDialog = false
      if (!response) return

      await this.$store.dispatch('User/deleteUser', this.selectedUser.id)
    }
  },
  watch: {
    user: {
      immediate: true,
      handler (newUserValue, oldUserValue) {
        if (newUserValue != null) {
          this.fetchUsers()
        }
      }
    }
  }
}

export default UserList
</script>
