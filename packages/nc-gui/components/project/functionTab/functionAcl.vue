<template>
  <div>

    <v-toolbar flat height="42" class="toolbar-border-bottom">
      <v-toolbar-title>
        <v-breadcrumbs :items="[{
                text: this.nodes.env,
                disabled: true,
                href: '#'
              },{
                text: this.nodes.dbAlias,
                disabled: true,
                href: '#'
              },
              {
                text: (nodes.function_name) + ' (ACL)',
                disabled: true,
                href: '#'
              }]" divider=">" small>
          <template v-slot:divider>
            <v-icon small color="grey lighten-2">forward</v-icon>
          </template>
        </v-breadcrumbs>

      </v-toolbar-title>
      <v-spacer></v-spacer>
      <x-btn outlined tooltip="Reload ACL"
             color="primary"
             small
             @click="loadAcl"
             v-ge="['acl','reload']"
      >
        <v-icon small left>refresh</v-icon>
        Reload
      </x-btn>

      <x-btn outlined tooltip="Save Changes"
             color="primary"
             class="primary"
             small
             v-ge="['acl','save']"
             @click="save"
             :disabled="!edited"
      >
        <v-icon small left>save</v-icon>
        Save
      </x-btn>

    </v-toolbar>

    <v-simple-table v-slot:default>
      <thead>
      <tr>
        <th></th>
        <th v-for="role in roles" :key="role">{{ role }}</th>
      </tr>
      </thead>
      <tbody>
      <tr>
        <th>{{ nodes.function_name }}</th>
        <th v-for="role in roles" :key="role" class="permission-checkbox-container">
          <v-checkbox
            @change="edited=true"
            v-model="data[role]"
            dense
            hide-details
          />
        </th>
      </tr>
      </tbody>
    </v-simple-table>


  </div>
</template>

<script>
export default {
  name: "functionAcl",
  data: () => ({
    loading: false,
    edited: false,
    data: null
  }),
  async created() {
    await this.loadAcl()
  },
  methods: {
    async loadAcl() {
      this.loading = true;
      const result = await this.$store.dispatch('sqlMgr/ActSqlOp', [{
        env: this.nodes.env,
        dbAlias: this.nodes.dbAlias
      }, 'xcAclGet', {
        name: this.nodes.function_name
      }]);
      this.data = JSON.parse(result.acl);

      this.loading = false;
    },
    async save() {
      try {
        await this.$store.dispatch('sqlMgr/ActSqlOp', [{
          env: this.nodes.env,
          dbAlias: this.nodes.dbAlias
        }, 'xcAclSave', {
          name: this.nodes.function_name,
          acl: this.data
        }]);
        this.$toast.success('ACL saved successfully').goAway(3000);
        this.edited = false;
        await this.loadAcl()
      } catch (e) {
        this.$toast.error('Some error occurred').goAway(3000);
      }
    }
  },
  props: ['nodes'],
  computed: {
    roles() {
      return this.data ? Object.keys(this.data) : []
    }
  }
}
</script>

<style scoped lang="scss">

::v-deep {
  .permission-checkbox-container .v-input {
    transform: scale(.8);
    margin-top: 0;
  }

  .v-overlay__content {
    height: 100%;
    display: flex;
    align-items: center;
  }
}
</style>
<!--
/**
 * @copyright Copyright (c) 2021, Xgene Cloud Ltd
 *
 * @author Naveen MR <oof1lab@gmail.com>
 * @author Pranav C Balan <pranavxc@gmail.com>
 *
 * @license GNU AGPL version 3 or any later version
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU Affero General Public License as
 * published by the Free Software Foundation, either version 3 of the
 * License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU Affero General Public License for more details.
 *
 * You should have received a copy of the GNU Affero General Public License
 * along with this program. If not, see <http://www.gnu.org/licenses/>.
 *
 */
-->
