<template>
  <div class="bg-warning text-dark p-5">
    <h1>Contador Manual de Personas</h1>
    <p>Conteo actual: {{ count }}</p>
    <button @click="increment" class="btn btn-info">Incrementar</button>
    <button @click="decrement" class="btn btn-info ms-3">Decrementar</button>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { io } from 'socket.io-client';

const count = ref(0);
const localizacion = ref('');
let socket = null;

const conectarSocket = () => {
  if (!socket) {
    socket = io('https://ikcount.com/live', {
      transports: ['polling'],
      query: {
        appKey: 'JDJiJDEwJGNQelI0OEJRa2ZMbEZYYXQ2aGpNNU8zYmRBbWtTYS8zV090TjVydDQyMnFzMDBieTY3Y3R5OmNsaWNrZXIxLmRlbW86SUtMQUIwMDU='
      }
    });

    socket.on('connect', () => {
      console.log('Conexión establecida con el servidor Socket.IO');
    });

    socket.on('disconnect', () => {
      console.log('Desconectado del servidor Socket.IO');
    });

    socket.on('error', (error) => {
      console.error('Error en la conexión Socket.IO:', error);
    });

    socket.on('SUMMARY', (data) => {
      count.value = data.count;
      localizacion.value = data.localizacion;
    });
  }
};

onMounted(() => {
  conectarSocket();
});

onUnmounted(() => {
  if (socket) {
    socket.disconnect();
  }
});

const fetchCommand = async (commandType) => {
  try {
    const params = new URLSearchParams({
      appKey: 'JDJiJDEwJGNQelI0OEJRa2ZMbEZYYXQ2aGpNNU8zYmRBbWtTYS8zV090TjVydDQyMnFzMDBieTY3Y3R5OmNsaWNrZXIxLmRlbW86SUtMQUIwMDU='
    });

    const response = await fetch(`https://ikcount.com/iklab/ikcount/api/counting/command?${params.toString()}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        type: commandType,
        quantity: 1,
        client: 'DEMO001',
        location: 'DEMO001A1L1',
        mac_address: 'DEMO001A1L1Z1MC1'
      })
    });

    const data = await response.json();

    if (response.ok) {
      console.log('Comando ejecutado con éxito');
    } else {
      console.error('Error al ejecutar el comando:', data.error);
    }
  } catch (error) {
    console.error('Error en la solicitud:', error);
  }
};

const increment = () => {
  fetchCommand('manual-add');
};

const decrement = () => {
  fetchCommand('manual-sub');
};
</script>
