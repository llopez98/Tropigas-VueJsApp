<template>
  <v-container>

    <div class="mt-5 mb-10">
      <v-row align="center" justify="space-around">
        <v-btn color="primary" @click.prevent="setArticulos">
          <v-icon left>
            mdi-archive
          </v-icon>
          Articulos
        </v-btn>

        <v-btn color="success" @click.prevent="setRegistrar">
          <v-icon left>
            mdi-pencil
          </v-icon>
          Registrar Ventas
        </v-btn>

        <v-btn color="warning" @click.prevent="setEditar">
          <v-icon left>
            mdi-circle-edit-outline
          </v-icon>
          Eliminar Ventas
        </v-btn>
      </v-row>
    </div>


    <div v-if="vistaArticulo == true">
      <v-card>
        <v-card-title>
          <v-text-field v-model="search" append-icon="mdi-magnify" label="Search" single-line hide-details>
          </v-text-field>
        </v-card-title>
        <v-data-table :headers="headers" :items="articulos" :search="search"></v-data-table>
      </v-card>
    </div>
    <div v-else-if="vistaRegistro == true">
      <form>
        <v-select v-model="VentaD.articulo" :items="articulos" item-text="nombre" label="Articulo" required solo>
        </v-select>
        <v-text-field v-model="VentaD.cantidad" label="Cantidad" required solo type="number"></v-text-field>
        <v-text-field v-model="VentaD.precio" label="Precio" required solo type="number"></v-text-field>



        <v-btn class="mr-4" @click.prevent="agregar">
          Agregar a factura
        </v-btn>

        <v-card class="mt-5">
          <v-card-title>
            <v-text-field v-model="search" append-icon="mdi-magnify" label="Search" single-line hide-details>
            </v-text-field>
          </v-card-title>
          <v-data-table :headers="headersFactura" :items="VentasD" :search="search"></v-data-table>
        </v-card>

        <v-text-field v-model="Venta.fecha" label="Fecha" required solo class="mt-5"></v-text-field>
        <v-text-field v-model="Venta.importe" label="Importe" required solo type="number"></v-text-field>
        <v-text-field v-model="Venta.usuario" label="Usuario" required solo></v-text-field>
        <v-btn @click.prevent="crearFactura">
          Facturar
        </v-btn>
      </form>
    </div>
    <div v-else-if="vistaEditar == true">
      <v-card>
        <v-card-title>
          <v-text-field v-model="search" append-icon="mdi-magnify" label="Search" single-line hide-details>
          </v-text-field>
        </v-card-title>
        <v-data-table :headers="headerVentas" :items="ventasTotales" :search="search">
          <template v-slot:[`item.delete`]="{ item }">
            <v-btn class="mx-2" fab dark small danger @click.prevent="deleteItem(item)">
              <v-icon dark> mdi-delete </v-icon>
            </v-btn>
          </template>
        </v-data-table>
      </v-card>
    </div>
<v-alert
v-model="alertGood"
      dense
      text
      type="success"
    >
      Datos cargados correctamente!
    </v-alert>
    <v-alert
v-model="alertBad"
      dense
      text
      type="danger"
    >
      Error al traer los datos!
    </v-alert>
  </v-container>
</template>

<script>
import axios from 'axios';
export default {
  name: 'Main',

  data: () => ({
    alertGood: false,
    alertBad: false,
    cont: 1,
    cont2: 1,
    vistaArticulo: true,
    vistaRegistro: false,
    vistaEditar: false,
    Venta: {
      id: "",
      fecha: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),
      importe: "",
      usuario: "",
      enviado: "",
    },
    VentaD: {
      id: "",
      renglon: "",
      articulo: "",
      cantidad: "",
      precio: "",
      enviado: "",
      codigo_detalle: "",
    },
    VentasD: [],
    Configuracion: {},
    articulos: [],
    ventas: [],
    ventasd: [],
    search: '',
    headers: [
      {
        text: 'Identificador',
        align: 'start',
        filterable: false,
        value: 'id',
      },
      { text: 'Nombre', value: 'nombre' },
    ],
    headersFactura: [
      {
        text: 'Identificador',
        align: 'start',
        filterable: false,
        value: 'id'
      },
      { text: "Renglon", value: "renglon" },
      { text: "Articulo", value: "articulo" },
      { text: "Cantidad", value: "cantidad" },
      { text: "Precio", value: "precio" },
    ],
    headerVentas: [
      {
        text: 'Identificador',
        align: 'start',
        filterable: false,
        value: 'id'
      },
      {
        text: 'Fecha',
        value: 'fecha'
      },
      { text: 'Importe', value: 'importe' },
      { text: 'Usuario', value: 'usuario' },
      { text: 'Enviado', value: 'enviado' },
      {
        text: 'Eliminar', value: "delete",
        sortable: false,
      }
    ],
    ventasTotales: [],
  }),
  methods: {
    setArticulos() {
      this.vistaArticulo = true;
      this.vistaEditar = false;
      this.vistaRegistro = false;
      this.alertGood = false;
      this.alertBad = false;
    },
    setRegistrar() {
      this.vistaArticulo = false;
      this.vistaEditar = false;
      this.vistaRegistro = true;
      this.alertGood = false;
      this.alertBad = false;
    },
    setEditar() {
      this.vistaArticulo = false;
      this.vistaEditar = true;
      this.vistaRegistro = false;
      this.alertGood = false;
      this.alertBad = false;
      this.getVentas();
    },
    agregar() {
      this.VentaD.id = "get_articulos" + this.cont2;
      this.VentaD.renglon = this.cont;
      this.VentaD.enviado = true;
      this.cont = this.cont + 1;
      this.VentasD.push(this.VentaD);
      this.VentaD = {};
    },
    async crearFactura() {
      this.Venta.enviado = true;
      this.Venta.id = "get_articulos" + this.cont2;
      axios.post('https://appservice-webapp-tropigas.azurewebsites.net/api/Venta', this.Venta).then(function (response) {
        console.log(response);
              this.alertGood = true;
                getVentas();
      })
        .catch(function (error) {
          console.log(error);
                this.alertGood = false;
                      this.alertBad = true;


        });
      axios.post('https://appservice-webapp-tropigas.azurewebsites.net/api/VentaD', this.VentasD).then(function (response) {
        console.log(response);
              this.alertGood = true;

      })
        .catch(function (error) {
          console.log(error);
          this.alertGood = false;
                      this.alertBad = true;
        });
      this.cont2++;
    },
    async getRoutes() {
      const { data } = await axios.get('https://appservice-webapp-tropigas.azurewebsites.net/api/Configuracion');
      this.Configuracion = data;
      
    },
    async getArticulos() {
      const { data } = await axios.get('https://appservice-webapp-tropigas.azurewebsites.net/api/Articulos');
      this.articulos = data;
      this.alertGood = true;
    },
    async getVentas() {
      const { data } = await axios.get('https://appservice-webapp-tropigas.azurewebsites.net/api/Venta');
      this.ventasTotales = data;
            this.alertGood = true;

    },
    async deleteItem(item) {
      await axios.delete('https://appservice-webapp-tropigas.azurewebsites.net/api/Venta/' + item.id);
      this.getVentas();
    }
  },
  mounted() {
    this.getRoutes();
    this.getArticulos();
    this.getVentas();
  }
}
</script>
