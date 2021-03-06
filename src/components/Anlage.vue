<style>
#container {
  padding-top: 0;
}

.content.large {
  margin-left: 30%;
  padding-top: 20px;
}

.overview.large {
  border-right: solid 1px #ccc;
  height: 100%;
  position: fixed;
  width: 30%;
}
</style>

<template>
  <v-container fluid id="container">
    <v-row>
      <v-col
        :cols="12"
        lg="3"
        :class="{ large: $vuetify.breakpoint.lg, overview: true }"
      >
        <v-list dense>
          <v-list-item ripple style="cursor: pointer;" @click="scrollTo('top')">
            <v-list-item-icon>
              <v-icon>mdi-castle</v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title
                v-text="anlage.anlagenname"
              ></v-list-item-title>
            </v-list-item-content>
          </v-list-item>

          <v-list-item
            v-for="projekt in projekte"
            :key="projekt.id"
            ripple
            style="cursor: pointer;"
            @click="scrollTo(`projekt-${projekt.id}`)"
          >
            <v-list-item-icon>
              <v-icon>mdi-home-group</v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title
                v-text="projekt.projektname"
              ></v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-col>

      <v-col
        :cols="12"
        lg="8"
        :class="{ large: $vuetify.breakpoint.lg, content: true }"
      >
        <v-card ref="top">
          <DescriptionTable
            :item="anlage"
            :props="[
              { prop: 'anlagen_id' },
              {
                prop: 'kontaktperson',
                person: true,
              },
              { prop: 'beschreibung' },
              { prop: 'standort' },
              { prop: 'standortcode' },
              { prop: 'avanti_link', title: 'Link zu Avanti' },
            ]"
          >
            <template v-slot:prop.avanti_link="{ item }">
              <AvantiLink :item="item" />
            </template>
          </DescriptionTable>
        </v-card>

        <br />
        <hr />
        <br />

        <div
          v-for="projekt in projekte"
          v-bind:key="projekt.id"
          :ref="`projekt-${projekt.id}`"
        >
          <Projekt :projekt="projekt"></Projekt>

          <br />

          <v-data-iterator
            :items="filterByProp(objekte, 'projekt', projekt.id)"
            hide-default-footer
          >
            <template v-slot:no-data>
              <v-alert type="info">
                Keine Objekte im Projekt
              </v-alert>
            </template>
            <template v-slot:default="props">
              <v-row>
                <v-col
                  v-for="objekt in props.items"
                  :key="objekt.id"
                  cols="12"
                  sm="12"
                  md="6"
                  lg="6"
                >
                  <Objekt :objekt="objekt"></Objekt>
                </v-col>
                <template v-for="objekt in props.items">
                  <v-col
                    v-for="dienstleistung in filterByProp(
                      dienstleistungen,
                      'objekte',
                      objekt.id
                    )"
                    :key="dienstleistung.id"
                    cols="12"
                    sm="12"
                    md="6"
                    lg="6"
                  >
                    <Dienstleistung
                      :dienstleistung="dienstleistung"
                    ></Dienstleistung>
                  </v-col>
                </template>
              </v-row>
            </template>
          </v-data-iterator>

          <v-data-iterator
            :items="filterByProp(dienstleistungen, 'projekte', projekt.id)"
            hide-default-footer
          >
            <template v-slot:no-data
              ><v-alert type="info">
                Keine Dienstleistungen im Projekt.
                <a
                  href="https://limesurvey.bula21.ch/index.php/141511?newtest=Y&lang=de-informal"
                  target="_blank"
                  style="color: white;"
                >
                  <v-icon>mdi-cart</v-icon>
                  Dienstleistung bestellen
                </a>
              </v-alert></template
            >
            <template v-slot:default="props">
              <v-row>
                <v-col
                  v-for="item in props.items"
                  :key="item.id"
                  cols="12"
                  sm="12"
                  md="6"
                  lg="6"
                >
                  <Dienstleistung :dienstleistung="item"></Dienstleistung>
                </v-col>
              </v-row>
            </template>
          </v-data-iterator>

          <hr />
          <br />
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { apiAuthenticated, ApiError, filter } from "@/lib/api.js";
import { joinInPlace } from "@/lib/join.js";
import Dienstleistung from "@/components/Dienstleistung";
import DescriptionTable from "@/components/DescriptionTable";
import Objekt from "@/components/Objekt";
import Projekt from "@/components/Projekt";
import AvantiLink from "@/components/AvantiLink";

export default {
  name: "Anlage",
  components: {
    Dienstleistung,
    Projekt,
    DescriptionTable,
    Objekt,
    AvantiLink,
  },
  props: {
    anlage: {
      type: Object,
      default: () => {},
    },
    fields: {
      type: Object,
      default: () => {},
    },
    users: {
      type: Array,
      default: () => [],
    },
  },
  data: () => ({
    objekte: [],
    projekte: [],
    dienstleistungen: [],
  }),
  methods: {
    scrollTo(selector) {
      const target = this.$refs[selector];
      if (Array.isArray(target)) {
        this.$vuetify.goTo(target[0], { duration: 0 });
      } else {
        this.$vuetify.goTo(target, { duration: 0 });
      }
    },
    filterByProp: (objects, propName, propValue) =>
      objects.filter((obj) => obj[propName] === propValue),
    addFieldsInPlace(items, collectionName) {
      for (const item of items) {
        for (const [fieldName, fieldValue] of Object.entries(item)) {
          if (fieldValue === null) {
            continue;
          }
          const choices = this.fields[collectionName][fieldName]?.options
            ?.choices;
          if (choices !== undefined && choices[fieldValue] !== undefined) {
            item[fieldName] = choices[fieldValue];
          }
        }
      }
    },
    async fetchData() {
      try {
        // ressorts, projekte
        const [ressorts, projekte] = await Promise.all([
          apiAuthenticated("/items/ressort"),
          apiAuthenticated(
            "/items/projekt",
            filter("anlage", "=", this.anlage.id)
          ),
        ]);
        joinInPlace(projekte, ressorts, "ressort_betrieb");
        joinInPlace(projekte, this.users, "auftraggeber");
        joinInPlace(projekte, this.users, "verantwortliche_person_betrieb");
        this.addFieldsInPlace(projekte, "projekt");
        this.projekte = Object.freeze(projekte);

        const projekteIds = projekte.map((p) => p.id);

        // objekte
        const objekte = await apiAuthenticated(
          "/items/objekt",
          filter("projekt", "in", projekteIds)
        );
        joinInPlace(objekte, this.users, "kontaktperson_nutzung");
        joinInPlace(objekte, this.users, "kontaktperson_auftraggeber");
        joinInPlace(objekte, this.users, "planung");
        this.addFieldsInPlace(objekte, "objekt");
        this.objekte = Object.freeze(objekte);
        const objektIds = objekte.map((o) => o.id);

        // dienstleistungen
        const dienstleistungenProjekte = await apiAuthenticated(
          "/items/dienstleistung",
          filter("projekte", "in", projekteIds)
        );
        const dienstleistungenObjekte = await apiAuthenticated(
          "/items/dienstleistung",
          filter("objekte", "in", objektIds)
        );
        const dienstleistungenProjekteids = dienstleistungenProjekte.map(
          (p) => p.id
        );
        // merge dienstleistungen from projekte
        for (const d of dienstleistungenObjekte) {
          if (dienstleistungenProjekteids.indexOf(d.id) === -1) {
            dienstleistungenProjekte.push(d);
          }
        }
        joinInPlace(
          dienstleistungenProjekte,
          this.users,
          "kontaktperson_nutzung"
        );
        joinInPlace(
          dienstleistungenProjekte,
          this.users,
          "kontaktperson_auftraggeber"
        );
        this.addFieldsInPlace(dienstleistungenProjekte, "dienstleistung");
        this.dienstleistungen = Object.freeze(dienstleistungenProjekte);
      } catch (err) {
        if (err instanceof ApiError) {
          this.$emit("api-error", err.userMessage());
        } else {
          throw err;
        }
      }
    },
  },
  created() {
    this.fetchData();
  },
};
</script>
