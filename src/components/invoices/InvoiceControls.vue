<template>
    <div class="row" v-if="invoice">
        <div class="col-12 mb-4 d-flex justify-content-between align-items-start">
            <router-link class="btn btn-sm btn-light btn--icon-left"
                            :to="{name: 'invoices'}">
                <i class="material-icons">arrow_back</i>
                <span class="d-inline-block">Atrás</span>
            </router-link>
            <div class="d-flex align-items-center">
                <AppSelect :value="invoice.status"
                           class="mb-0 mr-2 text-capitalize multiselect--capitalize"
                           :options="['borrador', 'reservado', 'enviado', 'pagado', 'cancelado']"
                           @input="updateProp({status: $event})"/>
                <button class="btn btn-sm btn-outline-dark"
                        v-if="invoice.status === 'draft'"
                        @click="bookInvoice">Emitir
                </button>
                <b-dropdown variant="link" size="sm" no-caret right>
                    <template slot="button-content">
                        <i class="material-icons">more_vert</i>
                    </template>
                    <b-dropdown-item-button @click="print">Descargar PDF</b-dropdown-item-button>
                    <b-dropdown-item-button @click="deleteInvoice">Eliminar</b-dropdown-item-button>
                </b-dropdown>
            </div>
        </div>
    </div>
</template>

<script>
import { mapGetters } from 'vuex';
import NotificationService from '@/services/notification.service';
import { BDropdown, BDropdownItemButton } from 'bootstrap-vue';
import AppSelect from '@/components/form/AppSelect';

export default {
  components: {
    BDropdown,
    BDropdownItemButton,
    AppSelect,
  },
  computed: {
    ...mapGetters({
      invoice: 'invoices/invoice',
    }),
  },
  methods: {
    async deleteInvoice() {
      const confirmed = await this.$bvModal.msgBoxConfirm(`¿Borrar factura ${this.invoice.number}?`, {
        okTitle: 'Eliminar',
        okVariant: 'danger',
        cancelTitle: 'Cancelar',
        cancelVariant: 'btn-link',
        contentClass: 'bg-base dp--24',
      });
      if (confirmed) {
        await this.$store.dispatch('invoices/deleteInvoice', this.invoice);
        NotificationService.success('Eliminado');
        this.$router.push({
          name: 'invoices',
        });
      }
    },
    bookInvoice() {
      this.$store.dispatch('invoices/bookInvoice');
    },
    updateProp(props) {
      this.$store.dispatch('invoices/updateInvoice', props);
    },
    print() {
      window.print();
    },
  },
};
</script>
