<template>
<v-dialog v-model="dialog" persistent max-width="600px">
    <template v-slot:activator="{ on }">
        <v-tooltip bottom>
            <template v-slot:activator="onTool">
                <v-icon class="mx-5" v-on="{...on, ...onTool.on}">
                    mdi-pencil
                </v-icon>
            </template>
            <span>Ürünü Düzenle</span>
        </v-tooltip>
    </template>
    <v-card>
    <v-card-title>
        <span class="headline">Ürün Düzenleme Formu</span>
    </v-card-title>
    <v-card-text>
        <v-form
        ref="form"
        v-model="valid"
        lazy-validation
        >
        <v-container>
        <v-row>
            <v-col cols="12">
            <v-text-field :error-messages="error ? [error]: []" :rules="validation.name" label="Ürün Adı*" v-model.trim="form.name" required />
            </v-col>
            <v-col cols="6">
            <v-text-field :rules="validation.count" type="number" v-model.number="form.count" min="1" max="100" label="Ürün Adedi*" required />
            </v-col>
            <v-col cols="6">
            <v-text-field :rules="validation.type" type="text" v-model.trim="form.type" label="Ürün Tipi*" required />
            </v-col>
            <v-col cols="12">
            <v-textarea :rules="validation.description" rows="1" v-model.trim="form.description" no-resize single-line auto-grow label="Ürün Açıklaması*" required />
            </v-col>
        </v-row>
        </v-container>
        <small>* Tüm Alanlar Zorunlu</small>
        </v-form>
        <v-snackbar v-model="snackbar">
          {{ error }}
          <v-btn
            color="red"
            text
            @click.native="snackbar = false"
          >
            Kapat
          </v-btn>
        </v-snackbar>
    </v-card-text>
    <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="grey lighten-2" :disabled="loading" text @click.prevent.native="dialog = false">Vazgeç</v-btn>
        <v-btn color="red darken-1" :loading="loading" text @click.prevent.native="save">Kaydet</v-btn>
    </v-card-actions>
    </v-card>
</v-dialog>
</template>

<script>
export default {
  name: "Edit",
  props: {
      id: String,
      name: String,
      count: Number,
      type: String,
      description: String
  },
  data: () => ({
      dialog: false,
      valid: true,
      loading: false,
      snackbar: false,
      error: "",
      validation: {
        name: [
          v => !!v || 'Ürün Adı Gerekli !'
        ],
        type: [
          v => !!v || 'Ürün Tipi Gerekli !'
        ],
        description: [
          v => !!v || 'Ürün Açıklaması Gerekli !'
        ],
        count: [
          v => v > 0 || 'Ürün Adedi Gerekli !'
        ],
      },
      form: {
        name: "",
        count: 1,
        type: "",
        description: ""
      }
  }),
  watch: {
      dialog(e) {
          if(e) {
            this.form = {
                name: this.name,
                count: this.count,
                type: this.type,
                description: this.description
            }
            return
          }

            this.form = {
                name: "",
                count: 1,
                type: "",
                description: ""
            }
            this.error = ""
      }
  },
  mounted() {},
  methods: {
    save() {
      if (this.$refs.form.validate()) {
        this.loading = true
        this.$ipc.send("product:update", JSON.stringify({
          id: this.id,
          form: {
            name: this.form.name.toLocaleUpperCase(),
            count: this.form.count,
            type: this.form.type.toLocaleUpperCase(),
            description: this.form.description.toLocaleUpperCase()
          }
        }))
        this.$ipc.on("product:exits", (_, e) => { 
          this.error = e 
          this.loading = false
          this.snackbar = true  
        })
        this.$ipc.on("product:updated", () => {
          this.loading = false;
          this.dialog = false;
        })
      }
    }
  },
};
</script>
