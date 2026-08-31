<template>
  <!-- Pantalla de Carga (Spinner) -->
  <div v-if="cargando" class="pantalla-carga">
    <div class="spinner"></div>
    <p>Cargando sistema...</p>
  </div>

  <div v-else class="contenedor">
    <header class="encabezado">
      <div class="logo-titulo">
        <span class="icono-barberia">✂️</span>
        <div>
          <h1>Barbería Don Ramiro</h1>
          <p class="subtitulo">Gestión de servicios y clientes</p>
        </div>
      </div>
      <button @click="abrirNuevo" class="btn-nuevo">+ Registrar Servicio</button>
    </header>

    <div v-if="servicios.length > 0" class="resumen">
      <div class="resumen-item">
        <span class="resumen-numero">{{ servicios.length }}</span>
        <span class="resumen-etiqueta">Servicios</span>
      </div>
      <div class="resumen-item">
        <span class="resumen-numero">{{ contarPendientes() }}</span>
        <span class="resumen-etiqueta">Pendientes / Fiado</span>
      </div>
      <div class="resumen-item">
        <span class="resumen-numero">${{ calcularIngresos() }}</span>
        <span class="resumen-etiqueta">Ingresos (pagado)</span>
      </div>
    </div>

    <main>
      <div v-if="servicios.length === 0" class="mensaje-vacio">
        <span class="mensaje-vacio-icono">💈</span>
        <p>No hay servicios registrados. ¡Comienza a agregar clientes!</p>
      </div>

      <div class="grid-tarjetas">
        <div
          v-for="servicio in servicios"
          :key="servicio.id"
          class="tarjeta"
          :class="{ 'tarjeta-pendiente': servicio.estado !== 'Pagado' }"
        >
          <div class="tarjeta-cabecera">
            <h3>{{ servicio.cliente }}</h3>
            <span class="etiqueta-estado" :class="servicio.estado.toLowerCase()">
              {{ servicio.estado }}
            </span>
          </div>

          <div class="tarjeta-cuerpo">

            <div class="info-fila">
              <span class="info-etiqueta">Servicio</span>
              <span class="info-valor">{{ servicio.tipoServicio }}</span>
            </div>
            <div class="info-fila">
              <span class="info-etiqueta">Barbero</span>
              <span class="info-valor">{{ servicio.barbero }}</span>
            </div>

            <div class="contenedor-fecha-hora">
              <div class="cuadrito-info">
                <span class="icono-cuadrito"></span>
                <div class="dato-cuadrito">
                  <span class="etiqueta-mini">Fecha</span>
                  <span class="valor-mini">{{ servicio.fecha }}</span>
                </div>
              </div>
              <div class="cuadrito-info">
                <span class="icono-cuadrito"></span>
                <div class="dato-cuadrito">
                  <span class="etiqueta-mini">Hora</span>
                  <span class="valor-mini">{{ servicio.hora }}</span>
                </div>
              </div>
            </div>

            <div class="fila-precio-pago">
              <span class="precio-pill">${{ servicio.precio }}</span>
              <span class="metodo-pago-pill">
                <span v-if="servicio.metodoPago === 'Efectivo'">💵</span>
                <span v-else-if="servicio.metodoPago === 'Tarjeta'">💳</span>
                <span v-else-if="servicio.metodoPago === 'Transferencia'">📱</span>
                {{ servicio.metodoPago }}
              </span>
            </div>

            <div class="info-fila info-fila-calificacion">
              <span class="info-etiqueta">Calificación</span>
              <span class="info-valor estrellas">{{ '⭐'.repeat(servicio.calificacion) }}</span>
            </div>

            <p v-if="servicio.calificacion <= 2" class="alerta">
              Atención requerida (Baja calificación)
            </p>

            <p v-if="servicio.estado === 'Fiado'" class="alerta alerta-fiado">
               Cliente fiado — pendiente por cobrar
            </p>

            <p v-show="servicio.observaciones" class="observaciones-texto">
              <strong>Notas:</strong> {{ servicio.observaciones }}
            </p>
          </div>

          <div class="acciones-tarjeta">
            <button @click="editarServicio(servicio)" class="btn-editar">✏️ Editar</button>
            <button @click="confirmarEliminacion(servicio.id)" class="btn-eliminar">🗑️ Eliminar</button>
          </div>
        </div>
      </div>
    </main>

    <div v-show="mostrarModal" class="modal-fondo">
      <div class="modal-contenido">

        <div v-if="guardando" class="overlay-guardando">
          <div class="spinner spinner-chico"></div>
          <p>Guardando información del cliente...</p>
        </div>

        <h2>{{ editandoId ? 'Editar Servicio' : 'Nuevo Servicio' }}</h2>

        <p v-if="errorFormulario" class="mensaje-error"> {{ errorFormulario }}</p>

        <form @submit.prevent="guardarServicio">
          <fieldset :disabled="guardando" class="fieldset-form">
            <div class="grid-form">
              <div class="grupo-form">
                <label>Cliente:</label>
                <input type="text" v-model="form.cliente" required placeholder="Nombre del cliente">
              </div>

              <div class="grupo-form">
                <label>Servicio:</label>
                <select v-model="form.tipoServicio" @change="actualizarPrecio" required>
                  <option value="">Seleccione...</option>
                  <option>Corte clásico</option>
                  <option>Corte moderno</option>
                  <option>Barba</option>
                  <option>Corte + barba</option>
                  <option>Cejas</option>
                  <option>Tinte</option>
                </select>
              </div>

              <div class="grupo-form">
                <label>Barbero:</label>
                <select v-model="form.barbero" required>
                  <option value="">Seleccione...</option>
                  <option>Don Ramiro</option>
                  <option>Empleado 1</option>
                  <option>Empleado 2</option>
                </select>
              </div>

              <div class="grupo-form">
                <label>Precio ($):</label>
                <input type="number" v-model="form.precio" required min="0" placeholder="0">
              </div>

              <div class="grupo-form">
                <label>Fecha:</label>
                <input type="date" v-model="form.fecha" required>
              </div>

              <div class="grupo-form">
                <label>Hora:</label>
                <input type="time" v-model="form.hora" required>
              </div>

              <div class="grupo-form">
                <label>Método de pago:</label>
                <select v-model="form.metodoPago" required>
                  <option value="">Seleccione...</option>
                  <option>Efectivo</option>
                  <option>Tarjeta</option>
                  <option>Transferencia</option>
                </select>
              </div>

              <div class="grupo-form">
                <label>Estado de pago:</label>
                <select v-model="form.estado" required>
                  <option>Pagado</option>
                  <option>Pendiente</option>
                  <option>Fiado</option>
                </select>
              </div>

              <div class="grupo-form fila-completa">
                <label>Calificación del servicio:</label>
                <select v-model="form.calificacion" required>
                  <option value="5">5 - Excelente</option>
                  <option value="4">4 - Bueno</option>
                  <option value="3">3 - Regular</option>
                  <option value="2">2 - Malo</option>
                  <option value="1">1 - Pésimo</option>
                </select>
              </div>

              <div class="grupo-form fila-completa">
                <label>Observaciones:</label>
                <textarea v-model="form.observaciones" rows="2" placeholder="Detalles adicionales..."></textarea>
              </div>
            </div>

            <div class="acciones-form">
              <button type="button" @click="cerrarModal" class="btn-cancelar">Cancelar</button>
              <button type="submit" class="btn-guardar">
                {{ guardando ? 'Guardando...' : 'Guardar Registro' }}
              </button>
            </div>
          </fieldset>
        </form>
      </div>
    </div>

    <div v-show="mostrarModalEliminar" class="modal-fondo">
      <div class="modal-contenido modal-pequeno">
        <h2 class="titulo-peligro"> Confirmar Eliminación</h2>
        <p>¿Estás completamente seguro de que deseas eliminar este registro? Esta acción no se puede deshacer.</p>
        <div class="acciones-form">
          <button type="button" @click="mostrarModalEliminar = false" class="btn-cancelar">Cancelar</button>
          <button type="button" @click="ejecutarEliminacion" class="btn-eliminar-confirmar">Sí, eliminar</button>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('barberia_final', [])
const cargando = ref(true)
const mostrarModal = ref(false)
const mostrarModalEliminar = ref(false)
const editandoId = ref(null)
const idAEliminar = ref(null)
const guardando = ref(false) 
const errorFormulario = ref('') 

setTimeout(() => {
  cargando.value = false
}, 1500)

const listaPrecios = {
  'Corte clásico': 15000,
  'Corte moderno': 20000,
  'Barba': 10000,
  'Corte + barba': 25000,
  'Cejas': 5000,
  'Tinte': 35000
}

const form = ref({
  cliente: '',
  tipoServicio: '',
  barbero: '',
  fecha: '',
  hora: '',
  precio: '',
  metodoPago: '',
  estado: 'Pagado',
  calificacion: '5',
  observaciones: ''
})

function contarPendientes() {
  return servicios.value.filter(s => s.estado === 'Pendiente' || s.estado === 'Fiado').length
}

function calcularIngresos() {
  let total = 0
  for (const s of servicios.value) {
    if (s.estado === 'Pagado') {
      total += Number(s.precio || 0)
    }
  }
  return total.toLocaleString()
}

function actualizarPrecio() {
  if (listaPrecios[form.value.tipoServicio]) {
    form.value.precio = listaPrecios[form.value.tipoServicio]
  } else {
    form.value.precio = ''
  }
}

function formularioVacio() {
  return {
    cliente: '',
    tipoServicio: '',
    barbero: '',
    fecha: '',
    hora: '',
    precio: '',
    metodoPago: '',
    estado: 'Pagado',
    calificacion: '5',
    observaciones: ''
  }
}

function abrirNuevo() {
  form.value = formularioVacio()
  editandoId.value = null
  errorFormulario.value = ''
  mostrarModal.value = true
}

function editarServicio(servicio) {
  form.value = { ...servicio }
  editandoId.value = servicio.id
  errorFormulario.value = ''
  mostrarModal.value = true
}

function cerrarModal() {
  if (guardando.value) return 
  mostrarModal.value = false
  errorFormulario.value = ''
}


function confirmarEliminacion(id) {
  idAEliminar.value = id
  mostrarModalEliminar.value = true
}

function ejecutarEliminacion() {
  servicios.value = servicios.value.filter(s => s.id !== idAEliminar.value)
  mostrarModalEliminar.value = false
  idAEliminar.value = null
}


function validarFormulario() {
  if (!form.value.cliente || form.value.cliente.trim().length < 2) {
    return 'El nombre del cliente debe tener al menos 2 letras.'
  }
  if (!form.value.tipoServicio) {
    return 'Selecciona el tipo de servicio.'
  }
  if (!form.value.barbero) {
    return 'Selecciona qué barbero atendió.'
  }
  if (!form.value.fecha || !form.value.hora) {
    return 'La fecha y la hora son obligatorias.'
  }
  if (!form.value.precio || Number(form.value.precio) <= 0) {
    return 'El precio debe ser mayor a 0.'
  }
  if (!form.value.metodoPago) {
    return 'Selecciona el método de pago.'
  }
  return '' 
}


function guardarServicio() {
  const error = validarFormulario()
  if (error) {
    errorFormulario.value = error
    return
  }

  errorFormulario.value = ''
  guardando.value = true

  setTimeout(() => {
    if (editandoId.value) {
      const index = servicios.value.findIndex(s => s.id === editandoId.value)
      if (index !== -1) {
        servicios.value[index] = { ...form.value, id: editandoId.value }
      }
    } else {
      servicios.value.push({ ...form.value, id: Date.now() })
    }

    guardando.value = false
    mostrarModal.value = false
  }, 2000)
}
</script>

<style>

:root {
  --color-fondo: #f0f2f5;
  --color-primario: #2c3e50;
  --color-primario-claro: #34495e;
  --color-acento: #e67e22;
  --color-acento-claro: #f39c12;
  --color-exito: #27ae60;
  --color-peligro: #e74c3c;
  --color-borde: #dcdde1;
  --color-tarjeta: #ffffff;
  --sombra: 0 4px 6px rgba(0,0,0,0.05);
  --sombra-fuerte: 0 10px 25px rgba(0,0,0,0.15);
}

* { box-sizing: border-box; }

body {
  font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  background: linear-gradient(180deg, #eef1f5 0%, #e4e8ee 100%);
  margin: 0;
  padding: 0;
  color: #333;
  text-align: left; 
}

.contenedor {
  max-width: 1100px;
  margin: 0 auto;
  padding: 20px;
  text-align: left;
}

.pantalla-carga {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, var(--color-primario) 0%, #1a252f 100%);
  font-weight: bold;
  color: #fff;
}

.spinner {
  width: 50px;
  height: 50px;
  border: 5px solid rgba(255,255,255,0.25);
  border-top-color: var(--color-acento-claro);
  border-radius: 50%;
  animation: girar 1s linear infinite;
  margin-bottom: 15px;
}

.spinner-chico {
  width: 40px;
  height: 40px;
  border-width: 4px;
  border-color: #e0e0e0;
  border-top-color: var(--color-acento);
}

@keyframes girar {
  to { transform: rotate(360deg); }
}

.encabezado {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  background: linear-gradient(135deg, var(--color-primario) 0%, var(--color-primario-claro) 100%);
  color: white;
  padding: 20px 30px;
  border-radius: 14px;
  margin-bottom: 28px;
  box-shadow: 0 6px 16px rgba(44, 62, 80, 0.25);
}

.logo-titulo {
  display: flex;
  align-items: center;
  gap: 16px;
  text-align: left;
}

.icono-barberia {
  font-size: 2.2rem;
  flex-shrink: 0;
  filter: drop-shadow(0 2px 3px rgba(0,0,0,0.3));
}

.encabezado h1 {
  margin: 0;
  font-size: 1.4rem;
  line-height: 1.3;
  letter-spacing: 0.3px;
}

.subtitulo {
  margin: 4px 0 0;
  font-size: 0.85rem;
  color: rgba(255,255,255,0.7);
}

.btn-nuevo {
  flex-shrink: 0;
  background: linear-gradient(135deg, var(--color-acento) 0%, var(--color-acento-claro) 100%);
  color: white;
  border: none;
  padding: 12px 22px;
  cursor: pointer;
  font-weight: bold;
  border-radius: 10px;
  font-size: 0.95rem;
  white-space: nowrap;
  box-shadow: 0 4px 10px rgba(230, 126, 34, 0.35);
}
.btn-nuevo:hover { transform: translateY(-2px); box-shadow: 0 6px 14px rgba(230, 126, 34, 0.45); }
.btn-nuevo:active { transform: translateY(0); }

.resumen {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
  gap: 16px;
  margin-bottom: 28px;
}

.resumen-item {
  background: var(--color-tarjeta);
  border-radius: 12px;
  padding: 18px 20px;
  box-shadow: var(--sombra);
  border-left: 4px solid var(--color-acento);
  display: flex;
  flex-direction: column;
  gap: 4px;
  text-align: left;
}

.resumen-numero {
  font-size: 1.7rem;
  font-weight: 800;
  color: var(--color-primario);
  line-height: 1.1;
}

.resumen-etiqueta {
  font-size: 0.78rem;
  color: #7f8c8d;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

button {
  transition: all 0.2s ease;
  font-family: inherit;
}

.mensaje-vacio {
  text-align: center;
  padding: 60px 20px;
  color: #7f8c8d;
  background: var(--color-tarjeta);
  border-radius: 14px;
  border: 2px dashed var(--color-borde);
}
.mensaje-vacio-icono {
  display: block;
  font-size: 3rem;
  margin-bottom: 10px;
}

.grid-tarjetas {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(310px, 1fr));
  gap: 22px;
}

.tarjeta {
  background: var(--color-tarjeta);
  border-radius: 14px;
  padding: 22px;
  box-shadow: var(--sombra);
  border-top: 4px solid var(--color-primario);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  display: flex;
  flex-direction: column;
  text-align: left;
}
.tarjeta:hover {
  transform: translateY(-5px);
  box-shadow: var(--sombra-fuerte);
}

.tarjeta-pendiente {
  border-top: 4px solid var(--color-peligro);
  background-color: #fffafa;
}

.tarjeta-cabecera {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
  border-bottom: 1px solid var(--color-borde);
  padding-bottom: 14px;
  margin-bottom: 16px;
}

.tarjeta-cabecera h3 {
  margin: 0;
  color: var(--color-primario);
  font-size: 1.15rem;
}

.etiqueta-estado {
  padding: 5px 12px;
  border-radius: 20px;
  font-size: 0.72rem;
  font-weight: bold;
  text-transform: uppercase;
  letter-spacing: 0.3px;
  white-space: nowrap;
  flex-shrink: 0;
}
.etiqueta-estado.pagado { background: #d4edda; color: #155724; }
.etiqueta-estado.pendiente { background: #f8d7da; color: #721c24; }
.etiqueta-estado.fiado { background: #fff3cd; color: #856404; }

.tarjeta-cuerpo {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.info-fila {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  padding: 9px 0;
  border-bottom: 1px solid #f0f1f3;
}

.info-etiqueta {
  font-size: 0.78rem;
  font-weight: 700;
  color: #8a94a3;
  text-transform: uppercase;
  letter-spacing: 0.4px;
  flex-shrink: 0;
}

.info-valor {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--color-primario);
  text-align: right;
}

.info-fila-calificacion {
  border-bottom: none;
}

.estrellas {
  color: #f39c12;
  font-size: 0.95rem;
}

.contenedor-fecha-hora {
  display: flex;
  gap: 10px;
  margin: 14px 0;
}

.cuadrito-info {
  flex: 1;
  display: flex;
  align-items: center;
  gap: 10px;
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  padding: 12px;
  border-radius: 10px;
  border: 1px solid #dee2e6;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
}

.icono-cuadrito {
  font-size: 1.4rem;
  flex-shrink: 0;
}

.dato-cuadrito {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.etiqueta-mini {
  font-size: 0.68rem;
  color: #6c757d;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  font-weight: bold;
}

.valor-mini {
  font-size: 0.88rem;
  color: var(--color-primario);
  font-weight: 700;
}

.fila-precio-pago {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  margin: 14px 0 10px;
}

.precio-pill {
  background: #eafaf1;
  color: var(--color-exito);
  font-weight: 800;
  font-size: 1.15rem;
  padding: 8px 16px;
  border-radius: 10px;
  white-space: nowrap;
}

.metodo-pago-pill {
  background: #f4f5f7;
  color: var(--color-primario);
  font-weight: 600;
  font-size: 0.85rem;
  padding: 8px 14px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  gap: 6px;
  white-space: nowrap;
}

.alerta {
  color: #721c24;
  background: #f8d7da;
  padding: 10px 12px;
  border-radius: 8px;
  font-size: 0.85rem;
  font-weight: bold;
  margin: 10px 0 0;
}

.alerta-fiado {
  color: #856404;
  background: #fff3cd;
}

.observaciones-texto {
  font-style: italic;
  color: #666;
  background: #f1f2f6;
  padding: 10px 12px;
  border-radius: 8px;
  font-size: 0.88rem;
  margin: 10px 0 0;
}

.acciones-tarjeta {
  display: flex;
  gap: 10px;
  margin-top: 18px;
  padding-top: 16px;
  border-top: 1px solid #f0f1f3;
}

.btn-editar, .btn-eliminar {
  flex: 1;
  padding: 10px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  font-size: 0.88rem;
}
.btn-editar { background: #f1f2f6; color: var(--color-primario); }
.btn-editar:hover { background: #dfe4ea; }

.btn-eliminar { background: #ffeded; color: var(--color-peligro); }
.btn-eliminar:hover { background: #ffcccc; }

.modal-fondo {
  position: fixed;
  top: 0; left: 0; width: 100%; height: 100%;
  background: rgba(0,0,0,0.55);
  backdrop-filter: blur(4px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px;
}

.modal-contenido {
  position: relative;
  background: white;
  padding: 28px;
  border-radius: 14px;
  width: 100%;
  max-width: 600px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: var(--sombra-fuerte);
  text-align: left;
}

.modal-contenido h2 {
  margin-top: 0;
}

.modal-pequeno {
  max-width: 400px;
  text-align: center;
}

.titulo-peligro { color: var(--color-peligro); }

.mensaje-error {
  background: #f8d7da;
  color: #721c24;
  padding: 10px 14px;
  border-radius: 8px;
  font-size: 0.9rem;
  font-weight: bold;
  margin: 0 0 18px;
}

.overlay-guardando {
  position: absolute;
  inset: 0;
  background: rgba(255, 255, 255, 0.92);
  border-radius: 14px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 10;
  gap: 10px;
  font-weight: bold;
  color: var(--color-primario);
}

.fieldset-form {
  border: none;
  margin: 0;
  padding: 0;
}
.fieldset-form:disabled {
  opacity: 0.6;
}

.grid-form {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.grupo-form label {
  display: block;
  margin-bottom: 6px;
  font-size: 0.88rem;
  font-weight: bold;
  color: var(--color-primario);
}

.grupo-form input, .grupo-form select, .grupo-form textarea {
  width: 100%;
  padding: 10px;
  box-sizing: border-box;
  border: 1px solid var(--color-borde);
  border-radius: 8px;
  font-family: inherit;
  font-size: 0.92rem;
  transition: border-color 0.2s, box-shadow 0.2s;
}
.grupo-form input:focus, .grupo-form select:focus, .grupo-form textarea:focus {
  outline: none;
  border-color: var(--color-acento);
  box-shadow: 0 0 0 3px rgba(230, 126, 34, 0.15);
}

.fila-completa { grid-column: 1 / -1; }

.acciones-form {
  margin-top: 26px;
  display: flex;
  justify-content: flex-end;
  gap: 15px;
}

.btn-cancelar {
  background: #f1f2f6;
  color: #333;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 8px;
  font-weight: bold;
}
.btn-cancelar:hover { background: #dfe4ea; }

.btn-guardar {
  background: linear-gradient(135deg, var(--color-exito) 0%, #2ecc71 100%);
  color: white;
  border: none;
  padding: 10px 22px;
  cursor: pointer;
  border-radius: 8px;
  font-weight: bold;
  box-shadow: 0 4px 10px rgba(39, 174, 96, 0.35);
}
.btn-guardar:hover { transform: translateY(-1px); box-shadow: 0 6px 14px rgba(39, 174, 96, 0.45); }
.btn-guardar:disabled { cursor: not-allowed; opacity: 0.8; transform: none; }

.btn-eliminar-confirmar {
  background: var(--color-peligro);
  color: white;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 8px;
  font-weight: bold;
}
.btn-eliminar-confirmar:hover { background: #c0392b; }

@media (max-width: 600px) {
  .grid-form { grid-template-columns: 1fr; }
  .encabezado { flex-direction: column; align-items: stretch; text-align: center; }
  .logo-titulo { justify-content: center; }
  .btn-nuevo { width: 100%; }
  .contenedor-fecha-hora { flex-direction: column; }
  .fila-precio-pago { flex-direction: column; align-items: stretch; }
  .precio-pill, .metodo-pago-pill { text-align: center; justify-content: center; }
}
</style>