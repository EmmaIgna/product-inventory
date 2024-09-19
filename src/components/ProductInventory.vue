<template>
  <div>
    <!-- Input para filtrar productos por nombre -->
    <a-input
      v-model:value="searchText"
      placeholder="Buscar por nombre"
      style="margin-bottom: 16px;"
    />

    <!-- Tabla de productos -->
    <a-table
      :columns="columns"
      :data-source="filteredDataSource"
      bordered
      :pagination="paginationConfig"
      @change="handlePageSizeChange"
    >
      <template #bodyCell="{ column, text, record }">
        <template v-if="['name', 'price', 'quantity'].includes(column.dataIndex)">
          <div>
            <a-input
              v-if="editableData[record.key]"
              :type="column.dataIndex === 'price' || column.dataIndex === 'quantity' ? 'number' : 'text'"
              v-model:value="editableData[record.key][column.dataIndex]"
              @input="handleInput($event, column.dataIndex, record.key)"
              style="margin: -5px 0"
            />
            <template v-else>
              {{ text }}
            </template>
          </div>
        </template>
        <template v-else-if="column.dataIndex === 'operation'">
          <div class="editable-row-operations">
            <span v-if="editableData[record.key]">
              <a-typography-link @click="save(record.key)">Guardar</a-typography-link>
              <a-popconfirm title="¿Estás seguro de cancelar?" @confirm="cancel(record.key)">
                <a>Cancelar</a>
              </a-popconfirm>
              <a-popconfirm title="¿Estás seguro de eliminar?" @confirm="deleteProduct(record.key)">
                <a>Eliminar</a>
              </a-popconfirm>
            </span>
            <span v-else>
              <a @click="edit(record.key)">Editar</a>
              <a-popconfirm title="¿Estás seguro de eliminar?" @confirm="deleteProduct(record.key)">
                <a>Eliminar</a>
              </a-popconfirm>
            </span>
          </div>
        </template>
      </template>
    </a-table>

    <!-- Botón para agregar productos -->
    <a-button type="primary" @click="showModal">Agregar Producto</a-button>

    <!-- Componente del modal para agregar productos -->
    <AddProductModal v-model:open="isModalVisible" @addProduct="addProduct" />
  </div>
</template>

<script setup>
import { cloneDeep } from 'lodash-es';
import { reactive, ref, onMounted, computed } from 'vue';
import AddProductModal from './AddProductModal.vue'; 

const columns = [
  {
    title: 'Nombre del Producto',
    dataIndex: 'name',
    width: '40%',
    sorter: (a, b) => a.name.localeCompare(b.name),
  },
  {
    title: 'Precio',
    dataIndex: 'price',
    width: '20%',
    sorter: (a, b) => a.price - b.price,
  },
  {
    title: 'Cantidad',
    dataIndex: 'quantity',
    width: '20%',
    sorter: (a, b) => a.quantity - b.quantity,
  },
  {
    title: 'Operación',
    dataIndex: 'operation',
  },
];

const dataSource = ref([]);
const editableData = reactive({});
const isModalVisible = ref(false);
const searchText = ref('');

// Configuración de paginación
const paginationConfig = ref({
  pageSize: 10,  // Número de elementos por página
  showSizeChanger: true,  // Permite al usuario cambiar el número de elementos por página
  pageSizeOptions: ['10', '20', '50'],  // Opciones disponibles para el número de elementos por página
  showQuickJumper: true,  // Permite saltar a una página específica
});

const handlePageSizeChange = (event) => {
 paginationConfig.value = {
  pageSize: event.pageSize,  // Número de elementos por página
  showSizeChanger: true,  // Permite al usuario cambiar el número de elementos por página
  pageSizeOptions: ['10', '20', '50'],  // Opciones disponibles para el número de elementos por página
  showQuickJumper: true,  // Permite saltar a una página específica
}
};

// Función para cargar los productos desde localStorage
const loadProductsFromLocalStorage = () => {
  const storedProducts = localStorage.getItem('products');
  if (storedProducts) {
    dataSource.value = JSON.parse(storedProducts);
  } else {
    // Agregar ítems de prueba si no hay datos en localStorage
    dataSource.value = Array.from({ length: 30 }, (_, index) => ({
      key: String(index + 1),
      name: `Producto ${index + 1}`,
      price: (Math.random() * 100).toFixed(2),
      quantity: Math.floor(Math.random() * 100) + 1,
    }));
    saveProductsToLocalStorage();
  }
};

// Guardar productos en localStorage
const saveProductsToLocalStorage = () => {
  localStorage.setItem('products', JSON.stringify(dataSource.value));
};

// Función para editar un producto
const edit = key => {
  editableData[key] = cloneDeep(dataSource.value.find(item => key === item.key));
};

// Función para guardar los cambios
const save = key => {
  const product = dataSource.value.find(item => key === item.key);
  if (product) {
    Object.assign(product, editableData[key]);
    delete editableData[key];
    saveProductsToLocalStorage();
  }
};

// Función para cancelar la edición
const cancel = key => {
  delete editableData[key];
};

// Función para mostrar el modal
const showModal = () => {
  isModalVisible.value = true;
};

// Función para agregar un nuevo producto
const addProduct = (newProduct) => {
  dataSource.value.push({
    key: String(dataSource.value.length + 1),
    ...newProduct
  });

  saveProductsToLocalStorage();
  isModalVisible.value = false;
};

// Función para manejar la entrada de datos y filtrar caracteres no numéricos
const handleInput = (event, columnDataIndex, key) => {
  const value = event.target.value;
  const isNumberField = columnDataIndex === 'price' || columnDataIndex === 'quantity';
  if (isNumberField) {
    const filteredValue = value.replace(/[^0-9.]/g, '');
    editableData[key][columnDataIndex] = filteredValue;
  } else {
    editableData[key][columnDataIndex] = value;
  }
};

// Función para eliminar un producto
const deleteProduct = key => {
  dataSource.value = dataSource.value.filter(item => item.key !== key);
  saveProductsToLocalStorage();
};

// Computed property para filtrar los productos basándose en el texto de búsqueda
const filteredDataSource = computed(() => {
  if (!searchText.value) return dataSource.value;
  return dataSource.value.filter(product =>
    product.name.toLowerCase().includes(searchText.value.toLowerCase())
  );
});

// Cargar los productos desde localStorage cuando se monta el componente
onMounted(() => {
  loadProductsFromLocalStorage();
});
</script>

<style scoped>
.editable-row-operations a {
  margin-right: 8px;
}
</style>
