<template>
  <div>
    <v-simple-table dense v-if="propsFiltered.length > 0">
      <template v-slot:default>
        <tbody>
          <tr v-show="hasHiddenFields">
            <td>Mehr Felder anzeigen</td>
            <td>
              <v-switch
                v-show="!settings.showAllFields"
                v-model="showAllFields"
              ></v-switch>
            </td>
          </tr>

          <tr v-for="(prop, key) in propsFiltered" :key="key">
            <!-- key -->
            <td class="description" v-if="prop.title">{{ prop.title }}:</td>
            <td class="description" v-else>{{ capitalCase(prop.prop) }}:</td>

            <!-- value -->
            <td>
              <slot
                :name="`prop.${prop.prop}`"
                v-bind:item="item[prop.prop]"
                v-if="prop.person !== true"
              >
                <v-icon v-if="isEmpty(prop.prop)">mdi-diameter-variant</v-icon>
                <template v-else-if="typeof item[prop.prop] === 'boolean'">
                  <template v-if="item[prop.prop]"
                    ><v-icon>mdi-check-outline</v-icon></template
                  >
                  <template v-else><v-icon>mdi-close-outline</v-icon></template>
                </template>
                <template v-else>{{ item[prop.prop] }}</template>
              </slot>
              <Person v-else :item="item[prop.prop]" />
            </td>
          </tr>
        </tbody>
      </template>
    </v-simple-table>
    <v-alert v-else type="info">
      Keine Elemente
    </v-alert>
  </div>
</template>

<script>
import { capitalCase } from "capital-case";
import Person from "@/components/Person";
import { mapState } from "vuex";

export default {
  name: "DescriptionTable",
  components: {
    Person,
  },
  data: () => ({
    showAllFields: false,
  }),
  props: {
    item: Object,
    props: Array,
  },
  computed: {
    hasHiddenFields() {
      for (const prop of this.props) {
        if (prop.default_hide === true) {
          return true;
        }
      }
      return false;
    },
    propsFiltered() {
      return this.props.filter((prop) => {
        if (this.settings.hideEmpty && this.isEmpty(prop.prop)) {
          return false;
        }
        if (
          !this.settings.showAllFields &&
          !this.showAllFields &&
          prop.default_hide === true
        ) {
          return false;
        }
        return true;
      });
    },
    ...mapState({
      settings: "settings",
    }),
  },
  methods: {
    capitalCase,
    isEmpty(prop) {
      return (
        this.item[prop] === null ||
        this.item[prop] === undefined ||
        this.item[prop] === ""
      );
    },
  },
};
</script>

<style>
td.description {
  background-color: #eeeeee;
  font-weight: bold;
}
</style>
