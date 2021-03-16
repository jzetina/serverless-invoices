<template>
    <div>
        <div class="row">
            <div class="col-12">
                <h4>Cliente</h4>
            </div>
        </div>

        <div v-if="client" class="row">
            <div class="col-12">
                <h5>General</h5>
            </div>
            <AppInput :value="client.company_name" @change="updateProp({ company_name: $event })"
                      label="Company Name" field="company_name" :errors="errors" class="col-12"/>
            <AppInput :value="client.invoice_email" @change="updateProp({ invoice_email: $event })"
                      label="Email" field="invoice_email" :errors="errors" class="col-sm-7"/>
            <AppInput :value="client.company_reg_no" @change="updateProp({ company_reg_no: $event })"
                      label="Company reg no" field="company_reg_no" :errors="errors" class="col-sm-5"/>

            <div class="col-12">
                <h5>Configuración de Facturas</h5>
                <h6>Dirección</h6>
            </div>
            <AppInput :value="client.company_address" @change="updateProp({ company_address: $event })"
                      label="Dirección de la Empresa" field="company_address" :errors="errors"
                      class="col-12"/>
            <AppInput :value="client.company_postal_code"
                      @change="updateProp({ company_postal_code: $event })"
                      label="Código Postal" field="company_postal_code" :errors="errors"
                      class="col-sm-5"/>
            <AppInput :value="client.company_city" @change="updateProp({ company_city: $event })"
                      label="Ciudad" field="company_city" :errors="errors" class="col-sm-7"/>
            <AppInput :value="client.company_county" @change="updateProp({ company_county: $event })"
                      label="Estado" field="company_county" :errors="errors" class="col-sm-6"/>
            <AppInput :value="client.company_country" @change="updateProp({ company_country: $event })"
                      label="País" field="company_country" :errors="errors" class="col-sm-6"/>
            <AppInput :value="client.currency" @change="updateProp({ currency: $event })"
                      label="Moneda" field="currency" :errors="errors" class="col-sm-4"/>
            <AppInput :value="client.rate" @change="updateProp({ rate: $event })"
                      label="Costo por hora" field="rate" :errors="errors" class="col-sm-4"/>

            <div class="col-12">
                <h6>VAT</h6>
            </div>
            <AppInput :value="client.company_vat_no" @change="updateProp({ company_vat_no: $event })"
                      label="IVA de la Empresa" field="company_vat_no" :errors="errors" class="col-sm-8"/>
            <AppCheckbox :value="client.has_vat" @input="updateProp({ has_vat: $event })"
                         label="Aplicar IVA" field="has_vat" :errors="errors" class="col-sm-4"/>

            <div class="col-12">
                <h6>Detalles Bancarios</h6>
            </div>
            <AppSelect :value="client.bank_account"
                       track-by="id"
                       label="Cuenta Bancaria"
                       label-field="bank_name"
                       :options="bankAccounts || []"
                       @input="bankAccountChanged"
                       class="col-12"/>
        </div>
        <div v-if="!client">Cargando</div>

        <div class="row mt-3 text-right" v-if="client">
            <div class="col-12">
                <div v-if="!isNew">
                    <button class="btn btn-outline-danger mr-2" @click="deleteClient(client.id)">Eliminar</button>
                    <button class="btn btn-primary"
                            @click="$emit('done')">Hecho
                    </button>
                </div>
                <button v-else class="btn btn-primary ml-2"
                        :disabled="loading"
                        @click="createClient">Crear
                </button>
            </div>
        </div>
    </div>
</template>
<script>
import { mapGetters } from 'vuex';
import NotificationService from '@/services/notification.service';
import AppInput from '@/components/form/AppInput';
import AppSelect from '@/components/form/AppSelect';
import Errors from '@/utils/errors';
import AppCheckbox from '@/components/form/AppCheckbox';

export default {
  components: {
    AppCheckbox,
    AppInput,
    AppSelect,
  },
  data() {
    return {
      errors: new Errors(),
      loading: false,
    };
  },
  computed: {
    ...mapGetters({
      client: 'clients/client',
      bankAccounts: 'bankAccounts/all',
    }),
    isNew() {
      return this.client && this.client.$isNew;
    },
  },
  mounted() {
    this.getBankAccounts();
  },
  methods: {
    getBankAccounts() {
      this.$store.dispatch('bankAccounts/getBankAccounts');
    },
    updateProp(props) {
      if (this.isNew) {
        return this.$store.dispatch('clients/clientProps', props);
      }
      this.errors.clear();

      this.$store.dispatch('clients/updateClient', props)
        .then(() => {
          NotificationService.success('Actualizado');
        })
        .catch(err => this.errors.set(err.errors));
    },
    bankAccountChanged(val) {
      this.updateProp({
        bank_account_id: val ? val.id : null,
        bank_account: val,
      });
    },
    createClient() {
      this.loading = true;
      this.errors.clear();

      return this.$store.dispatch('clients/createNewClient', this.client)
        .then((client) => {
          this.$router.push({
            query: {
              clientId: client.id,
            },
          });
          this.$emit('done');
        })
        .catch(err => this.errors.set(err.errors))
        .finally(() => {
          this.loading = false;
        });
    },
    async deleteClient(clientId) {
      const confirmed = await this.$bvModal.msgBoxConfirm(`¿Eliminar cliente ${this.client.company_name}?`, {
        okTitle: 'Eliminar',
        okVariant: 'danger',
        cancelTitle: 'Cancelar',
        cancelVariant: 'btn-link',
        contentClass: 'bg-base dp--24',
      });
      if (confirmed) {
        this.$emit('done');
        await this.$store.dispatch('clients/deleteClient', clientId);
        try {
          NotificationService.success('Eliminado');
        } catch (err) {
          NotificationService.error(err.message);
        }
      }
    },
  },
};
</script>
