<template>
  <a-form
    :model="formState"
    name="productForm"
    :label-col="{ span: 12 }"
    :wrapper-col="{ span: 24 }"
    autocomplete="off"
    @finish="onFinish"
    @finishFailed="onFinishFailed"
    layout="vertical"
  >
    <a-form-item
      label="Nombre del Producto"
      name="name"
      :rules="[{ required: true, message: 'Por favor introduce el nombre del producto!' }]">
      <a-input v-model:value="formState.name" />
    </a-form-item>

    <a-form-item
      label="Precio"
      name="price"
      :rules="[{
        required: true, 
        message: 'Por favor introduce el precio!'
      }, {
        type: 'number', 
        min: 1, 
        message: 'El precio debe ser mayor a 0'
      }]">
      <a-input-number v-model:value="formState.price" min="1" style="width: 100%;" />
    </a-form-item>

    <a-form-item
      label="Cantidad"
      name="quantity"
      :rules="[{
        required: true, 
        message: 'Por favor introduce la cantidad!'
      }, {
        type: 'number', 
        min: 1, 
        message: 'La cantidad debe ser mayor a 0'
      }]">
      <a-input-number v-model:value="formState.quantity" min="1" style="width: 100%;" />
    </a-form-item>
    
    <a-form-item style="display: flex; flex-direction: column;">
      <a-button type="primary" html-type="submit" style="margin-right: 10px;">Guardar</a-button>
      <a-button @click="handleCancel">Limpiar</a-button>
    </a-form-item>
  </a-form>
</template>

<script setup>
import { reactive } from 'vue';
import { defineEmits } from 'vue';

const emit = defineEmits(['update:visible', 'addProduct']);

const formState = reactive({
  name: '',
  price: 0,
  quantity: 0,
});

const onFinish = values => {
  const productData = {
    ...values,
    price: Number(values.price),
    quantity: Number(values.quantity),
    key: Date.now().toString()
  };
  console.log('Producto agregado con éxito:', productData);
  emit('addProduct', productData); // Emitir el evento con los valores del formulario
  handleCancel(); // Opcional: cerrar el modal después de agregar el producto
};

const onFinishFailed = errorInfo => {
  console.log('Validación fallida:', errorInfo);
};

const handleCancel = () => {
  // Reiniciar el estado del formulario
  formState.name = '';
  formState.price = 0;
  formState.quantity = 0;

  // Emitir un evento para cerrar el modal
  emit('update:visible', false);
};
</script>
