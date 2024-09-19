<template>
    <a-modal 
      :open="visible" 
      title="Agregar Nuevo Producto" 
      @cancel="handleCancel"
      footer=""
    >
      <ProductForm @update:visible="handleCancel" @addProduct="handleAddProduct" />
    </a-modal>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  import ProductForm from './ProductForm.vue';
  import { defineEmits } from 'vue';
  
  const emit = defineEmits(['update:visible', 'addProduct']);
  
  const visible = ref(false);
  
  const handleCancel = () => {
    visible.value = false; // Cierra el modal cuando se cancela
    emit('update:visible', false); // Emitir evento para informar que se cerró el modal
  };
  
  const handleAddProduct = (newProduct) => {
    // Emitir el evento para agregar el producto a la lista en el componente principal
    emit('addProduct', newProduct);
    emit('update:visible', false); // Cerrar el modal después de agregar el producto
  };
  </script>
  