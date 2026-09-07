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
        <span class="resumen-etiqueta">Pendientes / Abonado</span>
      </div>
      <div class="resumen-item">
        <span class="resumen-numero">${{ calcularIngresos() }}</span>
        <span class="resumen-etiqueta">Ingresos (pagado)</span>
      </div>
    </div>

    <section v-if="clientesAbonados.length > 0" class="seccion-abonados">
      <h2 class="titulo-seccion">💰 Clientes Abonados</h2>
      <div class="lista-abonados">
        <div v-for="servicio in clientesAbonados" :key="servicio.id" class="fila-abonado">
          <span class="abonado-cliente">{{ servicio.cliente }}</span>
          <div class="abonado-datos">
            <div class="abonado-dato">
              <span class="abono-etiqueta">Abonado</span>
              <span class="abono-valor abono-pagado">{{ formatearMoneda(servicio.montoAbonado) }}</span>
            </div>
            <div class="abonado-dato">
              <span class="abono-etiqueta">Faltante</span>
              <span class="abono-valor abono-faltante">{{ formatearMoneda(calcularFaltante(servicio)) }}</span>
            </div>
            <div class="abonado-dato">
              <span class="abono-etiqueta">Total</span>
              <span class="abono-valor abono-total">{{ formatearMoneda(servicio.precio) }}</span>
            </div>
          </div>
        </div>
      </div>
    </section>

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

            <div class="fila-servicios">
              <span class="info-etiqueta">Servicios</span>
              <div class="tags-servicios">
                <span v-for="tipo in servicio.tiposServicio" :key="tipo" class="tag-servicio">{{ tipo }}</span>
              </div>
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
              <span class="precio-pill">{{ formatearMoneda(servicio.precio) }}</span>
              <span class="metodo-pago-pill">
                <span v-if="servicio.metodoPago === 'Efectivo'">💵</span>
                <span v-else-if="servicio.metodoPago === 'Tarjeta'">💳</span>
                <span v-else-if="servicio.metodoPago === 'Transferencia'">📱</span>
                {{ servicio.metodoPago }}
              </span>
            </div>

            <div class="info-fila info-fila-calificacion">
              <span class="info-etiqueta">Calificación</span>
              <span v-if="servicio.calificacion > 0" class="info-valor estrellas">{{ '⭐'.repeat(servicio.calificacion) }}</span>
              <span v-else class="info-valor sin-calificar">Sin calificar</span>
            </div>

            <p v-show="servicio.observaciones" class="observaciones-texto">
              <strong>Notas del cliente:</strong> {{ servicio.observaciones }}
            </p>

            <button v-if="servicio.calificacion === 0" @click="abrirCalificar(servicio)" class="btn-calificar">
              ⭐ Calificar este servicio
            </button>
            <button v-else @click="abrirCalificar(servicio)" class="btn-editar-calificacion">
              ✏️ Editar calificación
            </button>

            <p v-if="servicio.calificacion > 0 && servicio.calificacion <= 2" class="alerta">
              Atención requerida (Baja calificación)
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

        <form @submit.prevent="guardarServicio">
          <fieldset :disabled="guardando" class="fieldset-form">
            <div class="grid-form">
              <div class="grupo-form">
                <label>Cliente:</label>
                <input type="text" v-model="form.cliente" :class="{ 'campo-invalido': errores.cliente }" placeholder="Nombre del cliente">
                <p v-if="errores.cliente" class="error-campo">{{ errores.cliente }}</p>
              </div>

              <div class="grupo-form">
                <label>Barbero:</label>
                <select v-model="form.barbero" :class="{ 'campo-invalido': errores.barbero }">
                  <option value="">Seleccione...</option>
                  <option>Don Ramiro</option>
                  <option>Empleado 1</option>
                  <option>Empleado 2</option>
                </select>
                <p v-if="errores.barbero" class="error-campo">{{ errores.barbero }}</p>
              </div>

              <div class="grupo-form fila-completa">
                <label>Servicios (puede elegir varios):</label>
                <div class="dropdown-servicios">
                  <button type="button" class="dropdown-boton" :class="{ 'campo-invalido': errores.tiposServicio }" @click="alternarListaServicios">
                    <span v-if="form.tiposServicio.length === 0" class="dropdown-placeholder">Seleccione los servicios...</span>
                    <span v-else class="dropdown-resumen">{{ form.tiposServicio.join(', ') }}</span>
                    <span class="dropdown-flecha">{{ mostrarListaServicios ? '▲' : '▼' }}</span>
                  </button>

                  <div v-show="mostrarListaServicios" class="dropdown-lista">
                    <label class="dropdown-opcion" v-for="opcion in opcionesServicio" :key="opcion">
                      <input type="checkbox" :value="opcion" v-model="form.tiposServicio" @change="actualizarPrecio">
                      <span class="dropdown-opcion-nombre">{{ opcion }}</span>
                      <span class="dropdown-opcion-precio">${{ listaPrecios[opcion].toLocaleString() }}</span>
                    </label>
                    <button type="button" class="dropdown-cerrar" @click="mostrarListaServicios = false">Listo</button>
                  </div>
                </div>
                <p v-if="errores.tiposServicio" class="error-campo">{{ errores.tiposServicio }}</p>
              </div>

              <div class="grupo-form">
                <label>Precio total ($):</label>
                <input
                  type="text"
                  inputmode="numeric"
                  :value="formatearMoneda(form.precio)"
                  @input="actualizarPrecioManual"
                  :class="{ 'campo-invalido': errores.precio }"
                  placeholder="$ 0"
                >
                <p v-if="errores.precio" class="error-campo">{{ errores.precio }}</p>
              </div>

              <div class="grupo-form">
                <label>Fecha:</label>
                <input type="date" v-model="form.fecha" :class="{ 'campo-invalido': errores.fecha }">
                <p v-if="errores.fecha" class="error-campo">{{ errores.fecha }}</p>
              </div>

              <div class="grupo-form">
                <label>Hora:</label>
                <input type="time" v-model="form.hora" :class="{ 'campo-invalido': errores.hora }">
                <p v-if="errores.hora" class="error-campo">{{ errores.hora }}</p>
              </div>

              <div class="grupo-form">
                <label>Método de pago:</label>
                <select v-model="form.metodoPago" :class="{ 'campo-invalido': errores.metodoPago }">
                  <option value="">Seleccione...</option>
                  <option>Efectivo</option>
                  <option>Tarjeta</option>
                  <option>Transferencia</option>
                </select>
                <p v-if="errores.metodoPago" class="error-campo">{{ errores.metodoPago }}</p>
              </div>

              <div class="grupo-form">
                <label>Estado de pago:</label>
                <select v-model="form.estado" :class="{ 'campo-invalido': errores.estado }">
                  <option value="">Seleccione...</option>
                  <option>Pagado</option>
                  <option>Pendiente</option>
                  <option>Abonado</option>
                </select>
                <p v-if="errores.estado" class="error-campo">{{ errores.estado }}</p>
              </div>

              <div class="grupo-form" v-if="form.estado === 'Abonado'">
                <label>Monto abonado ($):</label>
                <input
                  type="text"
                  inputmode="numeric"
                  :value="formatearMoneda(form.montoAbonado)"
                  @input="actualizarMontoAbonadoManual"
                  :class="{ 'campo-invalido': errores.montoAbonado }"
                  placeholder="$ 0"
                >
                <p v-if="errores.montoAbonado" class="error-campo">{{ errores.montoAbonado }}</p>
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

    <div v-show="mostrarModalCalificar" class="modal-fondo">
      <div class="modal-contenido modal-pequeno">
        <h2>⭐ Calificar servicio</h2>
        <p class="texto-ayuda-calificar">Esta calificación y las observaciones las da el cliente, no el barbero.</p>

        <p v-if="errorCalificacion" class="mensaje-error">{{ errorCalificacion }}</p>

        <div class="grupo-form fila-completa">
          <label>Calificación del cliente:</label>
          <select v-model="formCalificacion.calificacion">
            <option value="0">Sin calificar</option>
            <option value="5">5 - Excelente</option>
            <option value="4">4 - Bueno</option>
            <option value="3">3 - Regular</option>
            <option value="2">2 - Malo</option>
            <option value="1">1 - Pésimo</option>
          </select>
        </div>

        <div class="grupo-form fila-completa">
          <label>Observaciones del cliente:</label>
          <textarea v-model="formCalificacion.observaciones" rows="3" placeholder="Comentarios que dejó el cliente..."></textarea>
        </div>

        <div class="acciones-form">
          <button type="button" @click="cerrarModalCalificar" class="btn-cancelar">Cancelar</button>
          <button type="button" @click="guardarCalificacion" class="btn-guardar">Guardar calificación</button>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('barberia_final', [])
const cargando = ref(true)
const mostrarModal = ref(false)
const mostrarModalEliminar = ref(false)
const mostrarModalCalificar = ref(false)
const editandoId = ref(null)
const idAEliminar = ref(null)
const calificandoId = ref(null)
const guardando = ref(false) 
const errores = ref({})
const errorCalificacion = ref('')
const mostrarListaServicios = ref(false)

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

const opcionesServicio = ['Corte clásico', 'Corte moderno', 'Barba', 'Corte + barba', 'Cejas', 'Tinte']

const HORA_MIN_ATENCION = 6
const HORA_MAX_ATENCION = 22

const form = ref({
  cliente: '',
  tiposServicio: [],
  barbero: '',
  fecha: '',
  hora: '',
  precio: '',
  metodoPago: '',
  estado: '',
  montoAbonado: ''
})

const formCalificacion = ref({
  calificacion: '0',
  observaciones: ''
})

const clientesAbonados = computed(() => {
  return servicios.value.filter(s => s.estado === 'Abonado')
})

function contarPendientes() {
  return servicios.value.filter(s => s.estado === 'Pendiente' || s.estado === 'Abonado').length
}

function calcularIngresos() {
  let total = 0
  for (const s of servicios.value) {
    if (s.estado === 'Pagado') {
      total += Number(s.precio || 0)
    } else if (s.estado === 'Abonado') {
      total += Number(s.montoAbonado || 0)
    }
  }
  return total.toLocaleString()
}

function actualizarPrecio() {
  let total = 0
  for (const tipo of form.value.tiposServicio) {
    if (listaPrecios[tipo]) {
      total += listaPrecios[tipo]
    }
  }
  form.value.precio = total > 0 ? total : ''
}

function formatearMoneda(valor) {
  const numero = Number(valor)
  if (!numero || numero <= 0) return ''
  return '$ ' + numero.toLocaleString('es-CO')
}

function actualizarPrecioManual(evento) {
  const soloNumeros = evento.target.value.replace(/\D/g, '')
  form.value.precio = soloNumeros ? Number(soloNumeros) : ''
}

function actualizarMontoAbonadoManual(evento) {
  const soloNumeros = evento.target.value.replace(/\D/g, '')
  form.value.montoAbonado = soloNumeros ? Number(soloNumeros) : ''
}

function calcularFaltante(servicio) {
  const total = Number(servicio.precio || 0)
  const abonado = Number(servicio.montoAbonado || 0)
  const faltante = total - abonado
  return faltante > 0 ? faltante : 0
}

function formularioVacio() {
  return {
    cliente: '',
    tiposServicio: [],
    barbero: '',
    fecha: '',
    hora: '',
    precio: '',
    metodoPago: '',
    estado: '',
    montoAbonado: ''
  }
}

function alternarListaServicios() {
  mostrarListaServicios.value = !mostrarListaServicios.value
}

function abrirNuevo() {
  form.value = formularioVacio()
  editandoId.value = null
  errores.value = {}
  mostrarListaServicios.value = false
  mostrarModal.value = true
}

function editarServicio(servicio) {
  form.value = {
    cliente: servicio.cliente,
    tiposServicio: [...servicio.tiposServicio],
    barbero: servicio.barbero,
    fecha: servicio.fecha,
    hora: servicio.hora,
    precio: servicio.precio,
    metodoPago: servicio.metodoPago,
    estado: servicio.estado,
    montoAbonado: servicio.montoAbonado || ''
  }
  editandoId.value = servicio.id
  errores.value = {}
  mostrarListaServicios.value = false
  mostrarModal.value = true
}

function abrirCalificar(servicio) {
  formCalificacion.value = {
    calificacion: String(servicio.calificacion || 0),
    observaciones: servicio.observaciones || ''
  }
  calificandoId.value = servicio.id
  errorCalificacion.value = ''
  mostrarModalCalificar.value = true
}

function cerrarModalCalificar() {
  mostrarModalCalificar.value = false
  errorCalificacion.value = ''
}

function guardarCalificacion() {
  const index = servicios.value.findIndex(s => s.id === calificandoId.value)
  if (index !== -1) {
    servicios.value[index] = {
      ...servicios.value[index],
      calificacion: Number(formCalificacion.value.calificacion),
      observaciones: formCalificacion.value.observaciones
    }
  }
  mostrarModalCalificar.value = false
  calificandoId.value = null
}

function cerrarModal() {
  if (guardando.value) return 
  mostrarModal.value = false
  errores.value = {}
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
  const nuevosErrores = {}

  if (!form.value.cliente || form.value.cliente.trim().length < 2) {
    nuevosErrores.cliente = 'Falta el nombre del cliente (mínimo 2 letras).'
  }
  if (!form.value.tiposServicio || form.value.tiposServicio.length === 0) {
    nuevosErrores.tiposServicio = 'Selecciona al menos un tipo de servicio.'
  }
  if (!form.value.barbero) {
    nuevosErrores.barbero = 'Falta seleccionar el barbero.'
  }
  if (!form.value.fecha) {
    nuevosErrores.fecha = 'Falta la fecha.'
  }
  if (!form.value.hora) {
    nuevosErrores.hora = 'Falta la hora.'
  }

  if (form.value.fecha && form.value.hora) {
    const fechaHoraServicio = new Date(form.value.fecha + 'T' + form.value.hora)
    if (isNaN(fechaHoraServicio.getTime())) {
      nuevosErrores.fecha = 'La fecha no es válida.'
      nuevosErrores.hora = 'La hora no es válida.'
    } else {
      const ahora = new Date()
      ahora.setSeconds(0, 0)
      if (fechaHoraServicio.getTime() < ahora.getTime()) {
        nuevosErrores.fecha = 'Esta fecha y hora ya pasaron.'
        nuevosErrores.hora = 'Esta fecha y hora ya pasaron.'
      } else {
        const partesHora = form.value.hora.split(':')
        const horaDecimal = Number(partesHora[0]) + Number(partesHora[1]) / 60
        if (horaDecimal < HORA_MIN_ATENCION || horaDecimal > HORA_MAX_ATENCION) {
          nuevosErrores.hora = `Debe estar entre las ${HORA_MIN_ATENCION}:00 a.m. y las ${HORA_MAX_ATENCION - 12}:00 p.m.`
        }
      }
    }
  }

  if (!form.value.precio || Number(form.value.precio) <= 0) {
    nuevosErrores.precio = 'Falta el precio (mayor a 0).'
  }
  if (!form.value.metodoPago) {
    nuevosErrores.metodoPago = 'Falta seleccionar el método de pago.'
  }
  if (!form.value.estado) {
    nuevosErrores.estado = 'Falta seleccionar el estado de pago.'
  }

  if (form.value.estado === 'Abonado') {
    if (!form.value.montoAbonado || Number(form.value.montoAbonado) <= 0) {
      nuevosErrores.montoAbonado = 'Falta el monto abonado (mayor a 0).'
    } else if (form.value.precio && Number(form.value.montoAbonado) >= Number(form.value.precio)) {
      nuevosErrores.montoAbonado = 'El abono no puede ser igual o mayor al total. Si ya pagó todo, selecciona "Pagado".'
    }
  }

  errores.value = nuevosErrores
  return Object.keys(nuevosErrores).length === 0
}


function guardarServicio() {
  const esValido = validarFormulario()
  if (!esValido) {
    return
  }

  if (form.value.estado !== 'Abonado') {
    form.value.montoAbonado = ''
  }

  guardando.value = true

  setTimeout(() => {
    if (editandoId.value) {
      const index = servicios.value.findIndex(s => s.id === editandoId.value)
      if (index !== -1) {
        servicios.value[index] = { ...servicios.value[index], ...form.value, id: editandoId.value }
      }
    } else {
      servicios.value.push({ ...form.value, id: Date.now(), calificacion: 0, observaciones: '' })
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
  max-width: 1900px;
  margin: 0 auto;
  padding: 24px 40px;
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
.etiqueta-estado.abonado { background: #fff3cd; color: #856404; }

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

.seccion-abonados {
  background: linear-gradient(135deg, #fff8e1 0%, #fdf3d7 100%);
  border: 1px solid #f0d798;
  border-radius: 14px;
  padding: 20px 24px;
  margin-bottom: 28px;
  box-shadow: var(--sombra);
}

.titulo-seccion {
  margin: 0 0 14px;
  font-size: 1.05rem;
  color: #8a6d1a;
}

.lista-abonados {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.fila-abonado {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  background: #ffffff;
  border-radius: 10px;
  padding: 12px 16px;
  border: 1px solid #f0e2b6;
}

.abonado-cliente {
  font-weight: 800;
  color: var(--color-primario);
  font-size: 0.98rem;
  flex-shrink: 0;
}

.abonado-datos {
  display: flex;
  gap: 22px;
  flex-wrap: wrap;
}

.abonado-dato {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 2px;
  min-width: 80px;
}

.abono-etiqueta {
  font-size: 0.7rem;
  font-weight: 700;
  color: #8a94a3;
  text-transform: uppercase;
  letter-spacing: 0.4px;
}

.abono-valor {
  font-weight: 800;
  font-size: 0.95rem;
}

.abono-pagado { color: var(--color-exito); }
.abono-faltante { color: var(--color-peligro); }
.abono-total { color: var(--color-primario); }

@media (max-width: 600px) {
  .fila-abonado { flex-direction: column; align-items: flex-start; }
  .abonado-datos { width: 100%; justify-content: space-between; gap: 10px; }
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

.campo-invalido {
  border-color: var(--color-peligro) !important;
  background: #fff5f5;
}
.campo-invalido:focus {
  box-shadow: 0 0 0 3px rgba(231, 76, 60, 0.18) !important;
}

.error-campo {
  color: var(--color-peligro);
  font-size: 0.78rem;
  font-weight: 600;
  margin: 5px 0 0;
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
  .contenedor { padding: 14px; }
  .grid-form { grid-template-columns: 1fr; }
  .encabezado { flex-direction: column; align-items: stretch; text-align: center; }
  .logo-titulo { justify-content: center; }
  .btn-nuevo { width: 100%; }
  .contenedor-fecha-hora { flex-direction: column; }
  .fila-precio-pago { flex-direction: column; align-items: stretch; }
  .precio-pill, .metodo-pago-pill { text-align: center; justify-content: center; }
}

.fila-servicios {
  padding: 9px 0;
  border-bottom: 1px solid #f0f1f3;
}

.fila-servicios .info-etiqueta {
  display: block;
  margin-bottom: 8px;
}

.tags-servicios {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.tag-servicio {
  background: #eef1f5;
  color: var(--color-primario);
  font-size: 0.78rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  white-space: nowrap;
}

.sin-calificar {
  color: #adb5bd;
  font-style: italic;
  font-weight: 600;
}

.btn-calificar {
  width: 100%;
  margin-top: 10px;
  background: #fff8e1;
  color: #b8860b;
  border: 1px dashed #e6c35c;
  padding: 9px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  font-size: 0.85rem;
}
.btn-calificar:hover { background: #fef3cd; }

.btn-editar-calificacion {
  width: 100%;
  margin-top: 10px;
  background: none;
  color: #7f8c8d;
  border: none;
  padding: 4px;
  cursor: pointer;
  font-weight: 600;
  font-size: 0.78rem;
  text-decoration: underline;
}
.btn-editar-calificacion:hover { color: var(--color-primario); }

.observaciones-texto {
  font-style: italic;
  color: #666;
  background: #f1f2f6;
  padding: 10px 12px;
  border-radius: 8px;
  font-size: 0.88rem;
  margin: 10px 0 0;
}

.texto-ayuda-calificar {
  color: #7f8c8d;
  font-size: 0.85rem;
  margin-top: -6px;
  margin-bottom: 18px;
}

.dropdown-servicios {
  position: relative;
}

.dropdown-boton {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
  padding: 10px;
  background: white;
  border: 1px solid var(--color-borde);
  border-radius: 8px;
  font-family: inherit;
  font-size: 0.92rem;
  cursor: pointer;
  text-align: left;
}
.dropdown-boton:hover { border-color: var(--color-acento); }

.dropdown-placeholder { color: #9aa1ab; }

.dropdown-resumen {
  color: var(--color-primario);
  font-weight: 600;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.dropdown-flecha {
  flex-shrink: 0;
  color: var(--color-acento);
  font-size: 0.75rem;
}

.dropdown-lista {
  position: absolute;
  top: calc(100% + 6px);
  left: 0;
  right: 0;
  background: white;
  border: 1px solid var(--color-borde);
  border-radius: 10px;
  box-shadow: var(--sombra-fuerte);
  padding: 8px;
  z-index: 20;
  max-height: 260px;
  overflow-y: auto;
}

.dropdown-opcion {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 8px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: normal;
}
.dropdown-opcion:hover { background: #f4f5f7; }
.dropdown-opcion input { width: auto; flex-shrink: 0; }

.dropdown-opcion-nombre { flex: 1; font-size: 0.9rem; color: #333; }
.dropdown-opcion-precio { font-size: 0.82rem; color: #7f8c8d; font-weight: 600; }

.dropdown-cerrar {
  width: 100%;
  margin-top: 6px;
  padding: 9px;
  background: var(--color-primario);
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  font-size: 0.85rem;
}
.dropdown-cerrar:hover { background: var(--color-primario-claro); }
</style>