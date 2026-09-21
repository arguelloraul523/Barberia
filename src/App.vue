<template>
<<<<<<< HEAD
  <div class="app-container">
    <!-- ENCABEZADO -->
    <header class="barber-header">
      <div class="brand-section">
        <div class="barber-pole-icon">💈</div>
        <div>
          <h1 class="brand-title">Barbería Don Ramiro</h1>
          <p class="brand-subtitle">Panel de Control & Registro Diario de Servicios</p>
        </div>
      </div>

      <div class="header-actions">
        <button @click="abrirNuevoModal" class="btn-primary-glow">
          <span class="btn-icon">✂️</span> Registrar Nuevo Servicio
        </button>
      </div>
    </header>

    <!-- ESTADÍSTICAS -->
    <section class="stats-dashboard">
      <div class="stat-card">
        <div class="stat-icon income-icon">💵</div>
        <div class="stat-info">
          <span class="stat-value">${{ calcularIngresosTotales().toLocaleString('es-CO') }}</span>
          <span class="stat-label">Ingresos Recaudados</span>
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-icon count-icon">📋</div>
        <div class="stat-info">
          <span class="stat-value">{{ servicios.length }}</span>
          <span class="stat-label">Registros de Servicios</span>
        </div>
      </div>

      <div class="stat-card" :class="{ 'stat-warning': contarAbonados() > 0 }">
        <div class="stat-icon debt-icon">💰</div>
        <div class="stat-info">
          <span class="stat-value">{{ contarAbonados() }}</span>
          <span class="stat-label">Servicios Abonados</span>
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-icon star-icon">⭐</div>
        <div class="stat-info">
          <span class="stat-value">{{ calcularPromedioCalificacion() }} / 5</span>
          <span class="stat-label">Calificación Promedio</span>
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-icon top-barber-icon">🏆</div>
        <div class="stat-info">
          <span class="stat-value small-text">{{ obtenerBarberoDestacado() }}</span>
          <span class="stat-label">Barbero Destacado</span>
        </div>
      </div>
    </section>

    <!-- RESUMEN DEL ÚLTIMO ABONO -->
    <section v-if="obtenerUltimoAbono()" class="payment-summary">
      <div class="payment-summary-title">
        <span>💳</span>
        <div>
          <h3>Último abono registrado</h3>
          <p>Información del pago realizado</p>
        </div>
      </div>

      <div class="payment-summary-data">
        <div>
          <span>Quién abonó</span>
          <strong>{{ obtenerUltimoAbono().cliente }}</strong>
        </div>
        <div>
          <span>Cuánto abonó</span>
          <strong>${{ Number(obtenerUltimoAbono().abono).toLocaleString('es-CO') }}</strong>
        </div>
        <div>
          <span>Total del servicio</span>
          <strong>${{ Number(obtenerUltimoAbono().total).toLocaleString('es-CO') }}</strong>
        </div>
        <div>
          <span>Saldo pendiente</span>
          <strong class="saldo-text">${{ Number(obtenerUltimoAbono().saldo).toLocaleString('es-CO') }}</strong>
        </div>
      </div>
    </section>

    <!-- LISTA DE ABONADOS -->
    <section v-if="obtenerListaAbonados().length > 0" class="debt-alert-panel abonados-panel">
      <div class="panel-header">
        <span class="panel-icon">🔔</span>
        <h3>Servicios con abono pendiente</h3>
      </div>

      <div class="debtors-grid">
        <div v-for="abonado in obtenerListaAbonados()" :key="abonado.cliente" class="debtor-card">
          <span class="debtor-name">👤 {{ abonado.cliente }}</span>
          <span class="debtor-amount">${{ abonado.saldo.toLocaleString('es-CO') }}</span>
          <span class="debtor-count">saldo pendiente</span>
        </div>
      </div>
    </section>

    <!-- BÚSQUEDA Y ORDEN -->
    <section class="controls-bar">
      <div class="search-box">
        <span class="search-icon">🔍</span>
        <input
          type="text"
          v-model="busquedaCliente"
          placeholder="Buscar cliente para ver historial..."
          class="input-search"
        />
        <button v-show="busquedaCliente" @click="busquedaCliente = ''" class="btn-clear-search">✖</button>
      </div>

      <div v-if="busquedaCliente.trim()" class="customer-history-badge">
        <span>
          Historial de <strong>{{ busquedaCliente }}</strong>:
          {{ contarServiciosCliente(busquedaCliente) }} visita(s) |
          Total pagado: ${{ calcularGastoTotalCliente(busquedaCliente).toLocaleString('es-CO') }}
        </span>
      </div>

      <div class="sort-buttons">
        <span class="sort-label">Ordenar por:</span>
        <button
          @click="criterioOrden = 'fecha-desc'"
          :class="['btn-sort', { active: criterioOrden === 'fecha-desc' }]"
        >
          📅 Más Recientes
        </button>
        <button
          @click="criterioOrden = 'precio-desc'"
          :class="['btn-sort', { active: criterioOrden === 'precio-desc' }]"
        >
          💰 Mayor Precio
        </button>
        <button
          @click="criterioOrden = 'calificacion-desc'"
          :class="['btn-sort', { active: criterioOrden === 'calificacion-desc' }]"
        >
          ⭐ Mejor Calificados
        </button>
      </div>
    </section>

    <!-- SERVICIOS REGISTRADOS -->
    <main class="main-content">
      <div v-if="obtenerServiciosFiltrados().length === 0" class="empty-state">
        <div class="empty-icon">✂️</div>
        <h3>No se encontraron registros de servicios</h3>
        <p v-if="busquedaCliente">No hay resultados para "{{ busquedaCliente }}".</p>
        <p v-else>Comienza registrando la primera atención del día con el botón de arriba.</p>
      </div>

      <div v-else class="services-grid">
        <div
          v-for="servicio in obtenerServiciosFiltrados()"
          :key="servicio.id"
          class="service-card"
          :class="{
            'card-pagado': servicio.estado === 'Pagado',
            'card-abonado': servicio.estado === 'Abonado'
          }"
        >
          <div class="shift-badge" :class="obtenerClaseTurno(servicio.hora)">
            {{ obtenerNombreTurno(servicio.hora) }}
          </div>

          <div class="card-header">
            <div class="client-details">
              <h3 class="client-name">{{ servicio.cliente }}</h3>

              <div class="service-tags">
                <span
                  v-for="nombreServicio in obtenerServiciosDelRegistro(servicio)"
                  :key="nombreServicio"
                  class="service-type-tag service-tag-card"
                >
                  💈 {{ nombreServicio }}
                </span>
              </div>
            </div>

            <span class="payment-status-pill" :class="servicio.estado.toLowerCase()">
=======
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
>>>>>>> c478cc73262e5a7342e9cf5bffc7c4942483dd10
              {{ servicio.estado }}
            </span>
          </div>

<<<<<<< HEAD
          <div class="card-body">
            <div class="info-row">
              <span class="info-label">Barbero:</span>
              <span class="info-value highlight-barber">💈 {{ servicio.barbero }}</span>
            </div>

            <div class="datetime-container">
              <div class="datetime-pill">
                <span class="icon">📅</span> {{ servicio.fecha }}
              </div>
              <div class="datetime-pill">
                <span class="icon">⏰</span> {{ servicio.hora }}
              </div>
            </div>

            <div class="price-payment-row">
              <div class="price-box">
                <span class="price-amount">${{ Number(servicio.precio).toLocaleString('es-CO') }}</span>
                <span v-if="Number(servicio.propina) > 0" class="tip-badge">
                  + ${{ Number(servicio.propina).toLocaleString('es-CO') }} propina
                </span>
              </div>

              <div class="method-pill">
                <span v-if="servicio.metodoPago === 'Efectivo'">💵</span>
                <span v-else-if="servicio.metodoPago === 'Transferencia'">📱</span>
                <span v-else-if="servicio.metodoPago === 'Tarjeta'">💳</span>
                <span>{{ servicio.metodoPago }}</span>
              </div>
            </div>

            <div v-if="servicio.estado === 'Abonado'" class="abono-card-detail">
              <div>
                <span>Abono</span>
                <strong>${{ Number(servicio.abono || 0).toLocaleString('es-CO') }}</strong>
              </div>
              <div>
                <span>Saldo</span>
                <strong>${{ calcularSaldoRegistro(servicio).toLocaleString('es-CO') }}</strong>
              </div>
            </div>

            <!-- CALIFICACIÓN SEPARADA DEL REGISTRO -->
            <div class="rating-section">
              <div class="rating-display">
                <span class="info-label">Calificación:</span>

                <span v-if="servicio.calificacion > 0" class="stars">
                  {{ '⭐'.repeat(servicio.calificacion) }}
                  <span class="rating-num">({{ servicio.calificacion }}/5)</span>
                </span>

                <span v-else class="no-rating">Sin calificar aún</span>
              </div>

              <p
                v-if="servicio.calificacion > 0 && servicio.calificacion <= 2"
                class="low-rating-warning"
              >
                ⚠️ Atención: Cliente insatisfecho
              </p>

              <button
                @click="abrirModalCalificar(servicio)"
                class="btn-rating-action"
              >
                {{ servicio.calificacion > 0 ? '✏️ Modificar Calificación' : '⭐ Añadir Calificación' }}
              </button>
            </div>

            <div v-show="servicio.observaciones" class="notes-box">
              <span class="notes-title">📝 Notas / Observaciones:</span>
              <p class="notes-text">{{ servicio.observaciones }}</p>
            </div>
          </div>

          <div class="card-actions">
            <button @click="editarServicio(servicio)" class="btn-action-edit">✏️ Editar</button>
            <button @click="confirmarEliminacion(servicio.id)" class="btn-action-delete">🗑️ Eliminar</button>
=======
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
>>>>>>> c478cc73262e5a7342e9cf5bffc7c4942483dd10
          </div>
        </div>
      </div>
    </main>

<<<<<<< HEAD
    <!-- MODAL DE REGISTRO / EDICIÓN -->
    <div v-show="mostrarModal" class="modal-backdrop">
      <div class="modal-box modal-register">
        <div class="modal-header">
          <div>
            <h2>{{ editandoId ? '✏️ Editar Servicio' : '✂️ Registrar Servicio' }}</h2>
            <p class="modal-subtitle">Complete los datos del cliente y del servicio.</p>
          </div>
          <button @click="cerrarModal" class="btn-close-modal">✖</button>
        </div>

        <div v-if="alertaFrecuenteVisible" class="loyalty-alert">
          🎉 ¡Cliente Frecuente! Esta persona ha venido {{ conteoClienteActual }} veces.
        </div>

        <form @submit.prevent="guardarServicio" class="modal-form">
          <div class="form-grid">
            <!-- CLIENTE -->
            <div class="form-group full-width">
              <label>Nombre del Cliente:</label>
              <input
                type="text"
                v-model="form.cliente"
                @input="verificarClienteFrecuente"
                placeholder="Ej. Juan Pérez"
                :class="{ 'input-error': errores.cliente }"
              />
              <span v-if="errores.cliente" class="error-msg">{{ errores.cliente }}</span>
            </div>

            <!-- VARIOS SERVICIOS -->
            <div class="form-group full-width">
              <label>Seleccione los servicios:</label>

              <div class="services-selector">
                <label
                  v-for="item in catalogoServicios"
                  :key="item.nombre"
                  class="service-option"
                  :class="{ selected: form.serviciosSeleccionados.includes(item.nombre) }"
                >
                  <input
                    type="checkbox"
                    v-model="form.serviciosSeleccionados"
                    :value="item.nombre"
                  />
                  <span class="service-option-text">
                    <strong>{{ item.nombre }}</strong>
                    <span>${{ item.precio.toLocaleString('es-CO') }}</span>
                  </span>
                </label>
              </div>

              <span v-if="errores.servicios" class="error-msg">{{ errores.servicios }}</span>

              <div class="form-total-box">
                <span>Total de servicios:</span>
                <strong>${{ calcularTotalFormulario().toLocaleString('es-CO') }}</strong>
              </div>
            </div>

            <!-- BARBERO -->
            <div class="form-group">
              <label>Seleccione el barbero:</label>
              <select v-model="form.barbero" :class="{ 'input-error': errores.barbero }">
                <option value="">Seleccione el barbero...</option>
                <option v-for="b in listaBarberos" :key="b" :value="b">{{ b }}</option>
              </select>
              <span v-if="errores.barbero" class="error-msg">{{ errores.barbero }}</span>
            </div>

            <!-- FECHA -->
            <div class="form-group">
              <label>Seleccione la fecha:</label>
              <input
                type="date"
                v-model="form.fecha"
                :class="{ 'input-error': errores.fecha }"
              />
              <span v-if="errores.fecha" class="error-msg">{{ errores.fecha }}</span>
            </div>

            <!-- HORA -->
            <div class="form-group">
              <label>Seleccione la hora de atención:</label>
              <input
                type="time"
                v-model="form.hora"
                :class="{ 'input-error': errores.hora }"
              />
              <span class="form-help">Horario de atención: 8:00 a. m. a 8:00 p. m.</span>
              <span v-if="errores.hora" class="error-msg">{{ errores.hora }}</span>
            </div>

            <!-- PROPINA -->
            <div class="form-group">
              <label>Propina ($ opcional):</label>
              <input
                type="text"
                inputmode="numeric"
                v-model="form.propina"
                placeholder="Ej. 3000"
                :class="{ 'input-error': errores.propina }"
              />
              <span v-if="errores.propina" class="error-msg">{{ errores.propina }}</span>
            </div>

            <!-- MÉTODO DE PAGO -->
            <div class="form-group">
              <label>Seleccione el método de pago:</label>
              <select v-model="form.metodoPago" :class="{ 'input-error': errores.metodoPago }">
                <option value="">Seleccione el método de pago...</option>
                <option value="Efectivo">💵 Efectivo</option>
                <option value="Transferencia">📱 Transferencia</option>
                <option value="Tarjeta">💳 Tarjeta</option>
              </select>
              <span v-if="errores.metodoPago" class="error-msg">{{ errores.metodoPago }}</span>
            </div>

            <!-- ESTADO DEL PAGO -->
            <div class="form-group">
              <label>Seleccione el estado del pago:</label>
              <select v-model="form.estado" :class="{ 'input-error': errores.estado }">
                <option value="">Seleccione el estado del pago...</option>
                <option value="Pagado">✅ Pagado</option>
                <option value="Abonado">💰 Abonado</option>
              </select>
              <span v-if="errores.estado" class="error-msg">{{ errores.estado }}</span>
            </div>

            <!-- ABONO -->
            <div v-if="form.estado === 'Abonado'" class="form-group full-width abono-form-box">
              <label>¿Cuánto abonó el cliente?</label>
              <input
                type="text"
                inputmode="numeric"
                v-model="form.abono"
                placeholder="Ej. 10000"
                :class="{ 'input-error': errores.abono }"
              />

              <div class="abono-resumen">
                <div>
                  <span>Total:</span>
                  <strong>${{ calcularTotalConPropina().toLocaleString('es-CO') }}</strong>
                </div>
                <div>
                  <span>Abono:</span>
                  <strong>${{ Number(form.abono || 0).toLocaleString('es-CO') }}</strong>
                </div>
                <div>
                  <span>Saldo:</span>
                  <strong class="saldo-text">${{ calcularSaldoFormulario().toLocaleString('es-CO') }}</strong>
                </div>
              </div>

              <span v-if="errores.abono" class="error-msg">{{ errores.abono }}</span>
            </div>

            <!-- TOTAL -->
            <div class="form-group full-width">
              <div class="grand-total-box">
                <span>Total a pagar:</span>
                <strong>${{ calcularTotalConPropina().toLocaleString('es-CO') }}</strong>
              </div>
            </div>
          </div>

          <div class="form-footer">
            <button type="button" @click="cerrarModal" class="btn-cancel">Cancelar</button>
            <button type="submit" class="btn-save">
              {{ editandoId ? 'Actualizar Registro' : 'Guardar Servicio' }}
            </button>
          </div>
=======
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
>>>>>>> c478cc73262e5a7342e9cf5bffc7c4942483dd10
        </form>
      </div>
    </div>

<<<<<<< HEAD
    <!-- MODAL ELIMINAR -->
    <div v-show="mostrarModalEliminar" class="modal-backdrop">
      <div class="modal-box modal-small text-center">
        <div class="warning-icon">⚠️</div>
        <h3>Confirmar Eliminación</h3>
        <p>¿Seguro que deseas eliminar este registro de servicio? Esta acción no se puede deshacer.</p>

        <div class="modal-actions-center">
          <button @click="mostrarModalEliminar = false" class="btn-cancel">Cancelar</button>
          <button @click="ejecutarEliminacion" class="btn-confirm-delete">Sí, Eliminar</button>
=======
    <div v-show="mostrarModalEliminar" class="modal-fondo">
      <div class="modal-contenido modal-pequeno">
        <h2 class="titulo-peligro"> Confirmar Eliminación</h2>
        <p>¿Estás completamente seguro de que deseas eliminar este registro? Esta acción no se puede deshacer.</p>
        <div class="acciones-form">
          <button type="button" @click="mostrarModalEliminar = false" class="btn-cancelar">Cancelar</button>
          <button type="button" @click="ejecutarEliminacion" class="btn-eliminar-confirmar">Sí, eliminar</button>
>>>>>>> c478cc73262e5a7342e9cf5bffc7c4942483dd10
        </div>
      </div>
    </div>

<<<<<<< HEAD
    <!-- MODAL DE CALIFICACIÓN SEPARADO -->
    <div v-show="mostrarModalCalificar" class="modal-backdrop">
      <div class="modal-box modal-small">
        <div class="modal-header">
          <h3>⭐ Calificación & Comentarios</h3>
          <button @click="cerrarModalCalificar" class="btn-close-modal">✖</button>
        </div>

        <p class="help-text">La calificación se registra después del servicio.</p>

        <div class="rating-picker">
          <label>Calificación (1 a 5 estrellas):</label>

          <div class="star-buttons">
            <button
              v-for="star in [1, 2, 3, 4, 5]"
              :key="star"
              type="button"
              @click="formCalificacion.calificacion = star"
              :class="['star-btn', { active: formCalificacion.calificacion >= star }]"
            >
              ★
            </button>
          </div>

          <span class="rating-label-selected">{{ formCalificacion.calificacion }} de 5 Estrellas</span>
        </div>

        <div class="form-group">
          <label>Observaciones del Cliente:</label>
          <textarea
            v-model="formCalificacion.observaciones"
            rows="3"
            placeholder="Notas del servicio o retroalimentación..."
          ></textarea>
        </div>

        <div class="modal-actions-right">
          <button @click="cerrarModalCalificar" class="btn-cancel">Cancelar</button>
          <button @click="guardarCalificacion" class="btn-save">Guardar Calificación</button>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

// Persistencia únicamente con useLocalStorage
const servicios = useLocalStorage('don_ramiro_barberia_db', [])

// Estados de interfaz
const mostrarModal = ref(false)
const editandoId = ref(null)
const mostrarModalEliminar = ref(false)
const idAEliminar = ref(null)
const mostrarModalCalificar = ref(false)
const calificandoId = ref(null)

// Búsqueda y orden
const busquedaCliente = ref('')
const criterioOrden = ref('fecha-desc')
const alertaFrecuenteVisible = ref(false)
const conteoClienteActual = ref(0)

// Datos del negocio
const listaBarberos = ['Don Ramiro', 'Empleado 1', 'Empleado 2']

// Servicios reales de barbería con precios de ejemplo
const catalogoServicios = [
  { nombre: 'Corte clásico con tijera', precio: 18000 },
  { nombre: 'Low Fade', precio: 25000 },
  { nombre: 'Mid Fade', precio: 25000 },
  { nombre: 'High Fade', precio: 27000 },
  { nombre: 'Taper Fade', precio: 25000 },
  { nombre: 'Corte Buzz Cut', precio: 22000 },
  { nombre: 'Corte + Barba', precio: 35000 },
  { nombre: 'Arreglo y perfilado de barba', precio: 15000 },
  { nombre: 'Perfilado de cejas', precio: 7000 },
  { nombre: 'Diseño de línea', precio: 5000 }
]

// Formulario principal
const form = ref({
  cliente: '',
  serviciosSeleccionados: [],
  barbero: '',
  fecha: '',
  hora: '',
  propina: '',
  metodoPago: '',
  estado: '',
  abono: ''
})

// Formulario de calificación separado del registro
const formCalificacion = ref({
  calificacion: 5,
  observaciones: ''
})

// Errores manejados por JavaScript, sin validaciones HTML
const errores = ref({})

// -------------------- FECHA Y HORA --------------------

function obtenerFechaHoyLocal() {
  const hoy = new Date()
  const anio = hoy.getFullYear()
  const mes = String(hoy.getMonth() + 1).padStart(2, '0')
  const dia = String(hoy.getDate()).padStart(2, '0')
  return `${anio}-${mes}-${dia}`
}

function obtenerHoraActualLocal() {
  const hoy = new Date()
  return `${String(hoy.getHours()).padStart(2, '0')}:${String(hoy.getMinutes()).padStart(2, '0')}`
}

function obtenerHoraInicial() {
  const horaActual = obtenerHoraActualLocal()

  if (horaActual < '08:00') return '08:00'
  if (horaActual > '20:00') return '20:00'

  return horaActual
}

function fechaEsAnteriorAHoy(fecha) {
  return fecha < obtenerFechaHoyLocal()
}

function validarFechaYHora(err) {
  const fecha = form.value.fecha
  const hora = form.value.hora

  if (!fecha) {
    err.fecha = 'Seleccione la fecha del servicio.'
    return false
  }

  if (fechaEsAnteriorAHoy(fecha)) {
    err.fecha = 'No se puede seleccionar una fecha que ya pasó.'
    return false
  }

  if (!hora) {
    err.hora = 'Seleccione la hora del servicio.'
    return false
  }

  if (hora < '08:00' || hora > '20:00') {
    err.hora = 'El horario de atención es de 8:00 a. m. a 8:00 p. m.'
    return false
  }

  if (fecha === obtenerFechaHoyLocal() && hora < obtenerHoraActualLocal()) {
    err.hora = 'Para hoy no se puede registrar una hora que ya pasó.'
    return false
  }

  return true
}

// -------------------- SERVICIOS Y PAGOS --------------------

function calcularTotalFormulario() {
  let total = 0

  for (let i = 0; i < form.value.serviciosSeleccionados.length; i++) {
    const nombre = form.value.serviciosSeleccionados[i]

    for (let j = 0; j < catalogoServicios.length; j++) {
      if (catalogoServicios[j].nombre === nombre) {
        total += Number(catalogoServicios[j].precio)
      }
    }
  }

  return total
}

function calcularTotalConPropina() {
  const totalServicios = calcularTotalFormulario()
  const propina = Number(form.value.propina || 0)

  if (isNaN(propina) || propina < 0) return totalServicios
  return totalServicios + propina
}

function calcularSaldoFormulario() {
  const total = calcularTotalConPropina()
  const abono = Number(form.value.abono || 0)

  if (isNaN(abono) || abono < 0) return total
  return Math.max(total - abono, 0)
}

function calcularSaldoRegistro(servicio) {
  const total = Number(servicio.precio || 0) + Number(servicio.propina || 0)
  const abono = Number(servicio.abono || 0)
  return Math.max(total - abono, 0)
}

function obtenerServiciosDelRegistro(servicio) {
  if (servicio.servicios && servicio.servicios.length > 0) {
    return servicio.servicios
  }

  if (servicio.tipoServicio) {
    return [servicio.tipoServicio]
  }

  return []
}

function seleccionarPrecioSugerido() {
  // Se conserva como función normal para cumplir la estructura vista en clase.
  // El total se calcula directamente a partir de los servicios seleccionados.
  return calcularTotalFormulario()
}

// -------------------- ESTADÍSTICAS --------------------

function calcularIngresosTotales() {
  let total = 0

  for (let i = 0; i < servicios.value.length; i++) {
    const s = servicios.value[i]
    total += Number(s.abono || 0)

    if (s.estado === 'Pagado' && Number(s.abono || 0) === 0) {
      total += Number(s.precio || 0) + Number(s.propina || 0)
    }
  }

  return total
}

function contarAbonados() {
  let contador = 0

  for (let i = 0; i < servicios.value.length; i++) {
    if (servicios.value[i].estado === 'Abonado') {
      contador++
    }
  }

  return contador
}

function calcularPromedioCalificacion() {
  let suma = 0
  let calificados = 0

  for (let i = 0; i < servicios.value.length; i++) {
    const cal = Number(servicios.value[i].calificacion || 0)

    if (cal > 0) {
      suma += cal
      calificados++
    }
  }

  if (calificados === 0) return '0.0'
  return (suma / calificados).toFixed(1)
}

function obtenerBarberoDestacado() {
  if (servicios.value.length === 0) return 'Ninguno'

  const conteo = {}

  for (let i = 0; i < servicios.value.length; i++) {
    const b = servicios.value[i].barbero

    if (b) {
      conteo[b] = (conteo[b] || 0) + 1
    }
  }

  let maxBarbero = 'Ninguno'
  let maxCortes = 0

  for (const barbero in conteo) {
    if (conteo[barbero] > maxCortes) {
      maxCortes = conteo[barbero]
      maxBarbero = barbero
    }
  }

  return maxBarbero
}

function obtenerUltimoAbono() {
  for (let i = servicios.value.length - 1; i >= 0; i--) {
    if (servicios.value[i].estado === 'Abonado') {
      const servicio = servicios.value[i]

      return {
        cliente: servicio.cliente,
        abono: Number(servicio.abono || 0),
        total: Number(servicio.precio || 0) + Number(servicio.propina || 0),
        saldo: calcularSaldoRegistro(servicio)
      }
    }
  }

  return null
}

function obtenerListaAbonados() {
  const abonados = []

  for (let i = 0; i < servicios.value.length; i++) {
    const s = servicios.value[i]

    if (s.estado === 'Abonado') {
      abonados.push({
        cliente: s.cliente,
        saldo: calcularSaldoRegistro(s)
      })
    }
  }

  return abonados
}

function contarServiciosCliente(nombreCliente) {
  if (!nombreCliente) return 0

  const q = nombreCliente.toLowerCase().trim()
  let contador = 0

  for (let i = 0; i < servicios.value.length; i++) {
    if (servicios.value[i].cliente.toLowerCase().trim().includes(q)) {
      contador++
    }
  }

  return contador
}

function calcularGastoTotalCliente(nombreCliente) {
  if (!nombreCliente) return 0

  const q = nombreCliente.toLowerCase().trim()
  let total = 0

  for (let i = 0; i < servicios.value.length; i++) {
    const s = servicios.value[i]

    if (s.cliente.toLowerCase().trim().includes(q)) {
      total += Number(s.abono || 0)

      if (s.estado === 'Pagado' && Number(s.abono || 0) === 0) {
        total += Number(s.precio || 0) + Number(s.propina || 0)
      }
    }
  }

  return total
}

function obtenerServiciosFiltrados() {
  let lista = [...servicios.value]

  if (busquedaCliente.value.trim()) {
    const query = busquedaCliente.value.toLowerCase().trim()

    lista = lista.filter(s =>
      s.cliente.toLowerCase().includes(query)
    )
  }

  if (criterioOrden.value === 'fecha-desc') {
    lista.sort((a, b) =>
      new Date(b.fecha + 'T' + (b.hora || '00:00')) -
      new Date(a.fecha + 'T' + (a.hora || '00:00'))
    )
  } else if (criterioOrden.value === 'precio-desc') {
    lista.sort((a, b) =>
      Number(b.precio || 0) - Number(a.precio || 0)
    )
  } else if (criterioOrden.value === 'calificacion-desc') {
    lista.sort((a, b) =>
      Number(b.calificacion || 0) - Number(a.calificacion || 0)
    )
  }

  return lista
}

function obtenerNombreTurno(horaStr) {
  if (!horaStr) return 'Día'

  const horaNum = parseInt(horaStr.split(':')[0], 10)

  if (horaNum < 12) return 'Mañana'
  if (horaNum < 18) return 'Tarde'

  return 'Noche'
}

function obtenerClaseTurno(horaStr) {
  const turno = obtenerNombreTurno(horaStr)

  if (turno === 'Mañana') return 'shift-morning'
  if (turno === 'Tarde') return 'shift-afternoon'

  return 'shift-night'
}

// -------------------- FORMULARIO --------------------

function obtenerFormVacio() {
  return {
    cliente: '',
    serviciosSeleccionados: [],
    barbero: '',
    fecha: obtenerFechaHoyLocal(),
    hora: obtenerHoraInicial(),
    propina: '',
    metodoPago: '',
    estado: '',
    abono: ''
  }
}

function verificarClienteFrecuente() {
  if (form.value.cliente.trim().length >= 3) {
    const conteo = contarServiciosCliente(form.value.cliente)
    conteoClienteActual.value = conteo

    if (conteo >= 5) {
      alertaFrecuenteVisible.value = true
      return
    }
  }

  alertaFrecuenteVisible.value = false
}

function abrirNuevoModal() {
  form.value = obtenerFormVacio()
  editandoId.value = null
  errores.value = {}
  alertaFrecuenteVisible.value = false
=======
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
>>>>>>> c478cc73262e5a7342e9cf5bffc7c4942483dd10
  mostrarModal.value = true
}

function editarServicio(servicio) {
  form.value = {
    cliente: servicio.cliente,
<<<<<<< HEAD
    serviciosSeleccionados: servicio.servicios
      ? [...servicio.servicios]
      : (servicio.tipoServicio ? [servicio.tipoServicio] : []),
    barbero: servicio.barbero,
    fecha: servicio.fecha,
    hora: servicio.hora,
    propina: servicio.propina || '',
    metodoPago: servicio.metodoPago || '',
    estado: servicio.estado || '',
    abono: servicio.abono || ''
  }

  editandoId.value = servicio.id
  errores.value = {}
  verificarClienteFrecuente()
  mostrarModal.value = true
}

function cerrarModal() {
=======
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
>>>>>>> c478cc73262e5a7342e9cf5bffc7c4942483dd10
  mostrarModal.value = false
  errores.value = {}
}

<<<<<<< HEAD
function validarFormulario() {
  const err = {}

  if (!form.value.cliente || form.value.cliente.trim().length < 2) {
    err.cliente = 'Debe ingresar el nombre del cliente.'
  }

  if (!form.value.serviciosSeleccionados || form.value.serviciosSeleccionados.length === 0) {
    err.servicios = 'Debe seleccionar al menos un servicio.'
  }

  if (!form.value.barbero) {
    err.barbero = 'Seleccione el barbero que atendió.'
  }

  validarFechaYHora(err)

  const propina = Number(form.value.propina || 0)

  if (form.value.propina !== '' && (isNaN(propina) || propina < 0)) {
    err.propina = 'La propina debe ser un valor válido de 0 o mayor.'
  }

  if (!form.value.metodoPago) {
    err.metodoPago = 'Seleccione el método de pago.'
  }

  if (!form.value.estado) {
    err.estado = 'Seleccione el estado del pago.'
  }

  const total = calcularTotalConPropina()

  if (total <= 0) {
    err.servicios = 'Seleccione servicios para calcular el total.'
  }

  if (form.value.estado === 'Abonado') {
    const abono = Number(form.value.abono || 0)

    if (!form.value.abono || isNaN(abono) || abono <= 0) {
      err.abono = 'Ingrese cuánto abonó el cliente.'
    } else if (abono >= total) {
      err.abono = 'El abono debe ser menor que el total. Si paga todo, seleccione Pagado.'
    }
  }

  errores.value = err
  return Object.keys(err).length === 0
}

function guardarServicio() {
  if (!validarFormulario()) return

  const total = calcularTotalFormulario()
  const propina = Number(form.value.propina || 0)
  const totalConPropina = total + propina
  const abono = form.value.estado === 'Abonado'
    ? Number(form.value.abono)
    : totalConPropina

  const datos = {
    cliente: form.value.cliente.trim(),
    servicios: [...form.value.serviciosSeleccionados],
    tipoServicio: form.value.serviciosSeleccionados[0] || '',
    barbero: form.value.barbero,
    fecha: form.value.fecha,
    hora: form.value.hora,
    precio: total,
    propina: propina,
    metodoPago: form.value.metodoPago,
    estado: form.value.estado,
    abono: abono
  }

  if (editandoId.value) {
    const idx = servicios.value.findIndex(s => s.id === editandoId.value)

    if (idx !== -1) {
      servicios.value[idx] = {
        ...servicios.value[idx],
        ...datos
      }
    }
  } else {
    servicios.value.push({
      id: Date.now(),
      ...datos,
      calificacion: 0,
      observaciones: ''
    })
  }

  cerrarModal()
}

// -------------------- ELIMINACIÓN --------------------
=======
>>>>>>> c478cc73262e5a7342e9cf5bffc7c4942483dd10

function confirmarEliminacion(id) {
  idAEliminar.value = id
  mostrarModalEliminar.value = true
}

function ejecutarEliminacion() {
  servicios.value = servicios.value.filter(s => s.id !== idAEliminar.value)
  mostrarModalEliminar.value = false
  idAEliminar.value = null
}

<<<<<<< HEAD
// -------------------- CALIFICACIÓN SEPARADA --------------------

function abrirModalCalificar(servicio) {
  calificandoId.value = servicio.id

  formCalificacion.value = {
    calificacion: servicio.calificacion || 5,
    observaciones: servicio.observaciones || ''
  }

  mostrarModalCalificar.value = true
}

function cerrarModalCalificar() {
  mostrarModalCalificar.value = false
  calificandoId.value = null
}

function guardarCalificacion() {
  const idx = servicios.value.findIndex(s => s.id === calificandoId.value)

  if (idx !== -1) {
    servicios.value[idx].calificacion = Number(formCalificacion.value.calificacion)
    servicios.value[idx].observaciones = formCalificacion.value.observaciones.trim()
  }

  cerrarModalCalificar()
}
</script>
<style>

/* Reset y variables de diseño Full Screen */
:root {
  --bg-dark: #0f172a;
  --bg-card: #1e293b;
  --bg-card-hover: #334155;
  --accent-gold: #f59e0b;
  --accent-gold-hover: #d97706;
  --text-main: #f8fafc;
  --text-muted: #94a3b8;
  --border-color: #334155;
  --success-color: #10b981;
  --warning-color: #f59e0b;
  --danger-color: #ef4444;
  --shadow-main: 0 10px 25px -5px rgba(0, 0, 0, 0.5);
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Plus Jakarta Sans', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background-color: var(--bg-dark);
  color: var(--text-main);
  min-height: 100vh;
  width: 100vw;
  overflow-x: hidden;
}

.app-container {
  width: 100%;
  min-height: 100vh;
  padding: 20px 30px;
  display: flex;
  flex-direction: column;
  gap: 22px;
}

/* Header */
.barber-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: linear-gradient(135deg, #1e293b 0%, #0f172a 100%);
  border: 1px solid var(--border-color);
  border-radius: 16px;
  padding: 20px 28px;
  box-shadow: var(--shadow-main);
}

.brand-section {
  display: flex;
  align-items: center;
  gap: 16px;
}

.barber-pole-icon {
  font-size: 2.5rem;
  background: #334155;
  padding: 10px;
  border-radius: 12px;
}

.brand-title {
  font-size: 1.6rem;
  font-weight: 800;
  color: var(--accent-gold);
  letter-spacing: -0.5px;
}

.brand-subtitle {
  font-size: 0.88rem;
  color: var(--text-muted);
}

.btn-primary-glow {
  background: linear-gradient(135deg, var(--accent-gold) 0%, var(--accent-gold-hover) 100%);
  color: #0f172a;
  border: none;
  font-weight: 800;
  font-size: 0.95rem;
  padding: 14px 24px;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.25s ease;
  box-shadow: 0 4px 15px rgba(245, 158, 11, 0.3);
}

.btn-primary-glow:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(245, 158, 11, 0.45);
}

/* Dashboard de Estadísticas */
.stats-dashboard {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(210px, 1fr));
  gap: 16px;
}

.stat-card {
  background: var(--bg-card);
  border: 1px solid var(--border-color);
  border-radius: 14px;
  padding: 18px 20px;
  display: flex;
  align-items: center;
  gap: 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.stat-card.stat-warning {
  border-color: var(--danger-color);
  background: rgba(239, 68, 68, 0.08);
}

.stat-icon {
  font-size: 1.8rem;
  background: rgba(255, 255, 255, 0.05);
  padding: 12px;
  border-radius: 12px;
}

.stat-info {
  display: flex;
  flex-direction: column;
}

.stat-value {
  font-size: 1.35rem;
  font-weight: 800;
  color: var(--text-main);
}

.stat-value.small-text {
  font-size: 1.1rem;
}

.stat-label {
  font-size: 0.78rem;
  color: var(--text-muted);
=======

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
>>>>>>> c478cc73262e5a7342e9cf5bffc7c4942483dd10
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

<<<<<<< HEAD
/* Alerta de Deudores */
.debt-alert-panel {
  background: rgba(239, 68, 68, 0.12);
  border: 1px solid var(--danger-color);
  border-radius: 14px;
  padding: 16px 20px;
}

.panel-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 12px;
  color: #fca5a5;
}

.debtors-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}

.debtor-card {
  background: #1e293b;
  padding: 8px 14px;
  border-radius: 8px;
  font-size: 0.88rem;
  display: flex;
  align-items: center;
  gap: 8px;
  border: 1px solid #7f1d1d;
}

.debtor-amount {
  color: var(--danger-color);
  font-weight: 800;
}

.debtor-count {
  color: var(--text-muted);
  font-size: 0.75rem;
}

/* Barra de Controles y Búsqueda */
.controls-bar {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  background: var(--bg-card);
  padding: 14px 20px;
  border-radius: 14px;
  border: 1px solid var(--border-color);
}

.search-box {
  position: relative;
  display: flex;
  align-items: center;
  min-width: 280px;
  flex: 1;
}

.search-icon {
  position: absolute;
  left: 12px;
  color: var(--text-muted);
}

.input-search {
  width: 100%;
  padding: 10px 36px 10px 38px;
  background: #0f172a;
  border: 1px solid var(--border-color);
  border-radius: 10px;
  color: var(--text-main);
  font-size: 0.9rem;
}

.input-search:focus {
  outline: none;
  border-color: var(--accent-gold);
}

.btn-clear-search {
  position: absolute;
  right: 10px;
  background: none;
  border: none;
  color: var(--text-muted);
  cursor: pointer;
}

.customer-history-badge {
  background: rgba(245, 158, 11, 0.15);
  border: 1px solid var(--accent-gold);
  color: var(--accent-gold);
  padding: 8px 14px;
  border-radius: 8px;
  font-size: 0.85rem;
}

.sort-buttons {
  display: flex;
  align-items: center;
  gap: 8px;
}

.sort-label {
  font-size: 0.82rem;
  color: var(--text-muted);
}

.btn-sort {
  background: #0f172a;
  border: 1px solid var(--border-color);
  color: var(--text-muted);
  padding: 8px 14px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.82rem;
  transition: all 0.2s;
}

.btn-sort.active, .btn-sort:hover {
  background: var(--accent-gold);
  color: #0f172a;
  font-weight: bold;
  border-color: var(--accent-gold);
}

/* Rejilla de Tarjetas */
.services-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 20px;
}

.service-card {
  background: var(--bg-card);
  border: 1px solid var(--border-color);
  border-radius: 16px;
  padding: 22px;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.25);
  transition: transform 0.2s, border-color 0.2s;
}

.service-card:hover {
  transform: translateY(-4px);
}

.service-card.card-pagado { border-top: 4px solid var(--success-color); }
.service-card.card-pendiente { border-top: 4px solid var(--warning-color); }
.service-card.card-fiado { border-top: 4px solid var(--danger-color); }

.shift-badge {
  position: absolute;
  top: 12px;
  right: 14px;
  font-size: 0.7rem;
  font-weight: 800;
  text-transform: uppercase;
  padding: 3px 8px;
  border-radius: 6px;
}

.shift-morning { background: #0284c7; color: #fff; }
.shift-afternoon { background: #d97706; color: #fff; }
.shift-night { background: #4f46e5; color: #fff; }

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-top: 10px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--border-color);
}

.client-name {
  font-size: 1.2rem;
  font-weight: 800;
  color: var(--text-main);
}

.service-type-tag {
  display: inline-block;
  margin-top: 4px;
  font-size: 0.82rem;
  color: var(--accent-gold);
  font-weight: 600;
}

.payment-status-pill {
  font-size: 0.72rem;
  font-weight: 800;
  padding: 4px 10px;
  border-radius: 20px;
  text-transform: uppercase;
}

.payment-status-pill.pagado { background: rgba(16, 185, 129, 0.2); color: var(--success-color); }
.payment-status-pill.pendiente { background: rgba(245, 158, 11, 0.2); color: var(--warning-color); }
.payment-status-pill.fiado { background: rgba(239, 68, 68, 0.2); color: var(--danger-color); }

.card-body {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin: 14px 0;
}

.info-row {
  display: flex;
  justify-content: space-between;
  font-size: 0.88rem;
}

.info-label { color: var(--text-muted); }
.highlight-barber { font-weight: 700; color: var(--text-main); }

.datetime-container {
  display: flex;
  gap: 10px;
}

.datetime-pill {
  flex: 1;
  background: #0f172a;
  padding: 8px 10px;
  border-radius: 8px;
  font-size: 0.8rem;
  color: var(--text-muted);
  text-align: center;
  border: 1px solid var(--border-color);
}

.price-payment-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(15, 23, 42, 0.6);
  padding: 10px 14px;
  border-radius: 10px;
}

.price-amount {
  font-size: 1.25rem;
  font-weight: 800;
  color: var(--success-color);
}

.tip-badge {
  display: block;
  font-size: 0.75rem;
  color: var(--accent-gold);
}

.method-pill {
  font-size: 0.85rem;
  color: var(--text-main);
  background: #334155;
  padding: 4px 10px;
  border-radius: 6px;
}

.rating-section {
  background: #0f172a;
  padding: 10px 12px;
  border-radius: 10px;
  border: 1px dashed var(--border-color);
}

.rating-display {
  display: flex;
  justify-content: space-between;
  font-size: 0.85rem;
}

.stars { color: var(--accent-gold); }
.no-rating { color: var(--text-muted); font-style: italic; }

.low-rating-warning {
  color: var(--danger-color);
  font-size: 0.78rem;
  font-weight: bold;
  margin-top: 6px;
}

.btn-rating-action {
  width: 100%;
  margin-top: 8px;
  background: #1e293b;
  border: 1px solid var(--border-color);
  color: var(--text-main);
  padding: 6px;
  border-radius: 6px;
  font-size: 0.78rem;
  cursor: pointer;
  transition: background 0.2s;
}

.btn-rating-action:hover {
  background: #334155;
}

.notes-box {
  background: #0f172a;
  padding: 10px;
  border-radius: 8px;
  font-size: 0.82rem;
}

.notes-title { color: var(--text-muted); display: block; margin-bottom: 2px; }
.notes-text { color: var(--text-main); }

.card-actions {
  display: flex;
  gap: 10px;
  padding-top: 12px;
  border-top: 1px solid var(--border-color);
}

.btn-action-edit, .btn-action-delete {
  flex: 1;
  padding: 8px;
  border-radius: 8px;
  border: none;
  font-weight: 700;
  font-size: 0.85rem;
  cursor: pointer;
  transition: opacity 0.2s;
}

.btn-action-edit { background: #334155; color: var(--text-main); }
.btn-action-delete { background: rgba(239, 68, 68, 0.2); color: var(--danger-color); }

.btn-action-edit:hover, .btn-action-delete:hover { opacity: 0.85; }

/* Modales */
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.75);
  backdrop-filter: blur(4px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
  padding: 20px;
}

.modal-box {
  background: var(--bg-card);
  border: 1px solid var(--border-color);
  border-radius: 18px;
  width: 100%;
  max-width: 620px;
  padding: 26px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6);
  max-height: 90vh;
  overflow-y: auto;
}

.modal-small { max-width: 420px; }

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 18px;
}

.btn-close-modal {
  background: none;
  border: none;
  color: var(--text-muted);
  font-size: 1.2rem;
  cursor: pointer;
}

.loyalty-alert {
  background: rgba(245, 158, 11, 0.15);
  border: 1px solid var(--accent-gold);
  color: var(--accent-gold);
  padding: 12px;
  border-radius: 10px;
  font-size: 0.88rem;
  margin-bottom: 16px;
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 14px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-group.full-width { grid-column: 1 / -1; }

.form-group label {
  font-size: 0.82rem;
  color: var(--text-muted);
  font-weight: 700;
}

.form-group input, .form-group select, .form-group textarea {
  background: #0f172a;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  padding: 10px 12px;
  color: var(--text-main);
  font-size: 0.9rem;
  font-family: inherit;
}

.form-group input:focus, .form-group select:focus, .form-group textarea:focus {
  outline: none;
  border-color: var(--accent-gold);
}

.input-error {
  border-color: var(--danger-color) !important;
}

.error-msg {
  color: var(--danger-color);
  font-size: 0.75rem;
}

.form-footer, .modal-actions-right, .modal-actions-center {
  display: flex;
  gap: 12px;
  margin-top: 22px;
}

.form-footer, .modal-actions-right { justify-content: flex-end; }
.modal-actions-center { justify-content: center; }

.btn-cancel {
  background: #334155;
  color: var(--text-main);
  border: none;
  padding: 10px 18px;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
}

.btn-save {
  background: var(--accent-gold);
  color: #0f172a;
  border: none;
  padding: 10px 22px;
  border-radius: 8px;
  font-weight: 800;
  cursor: pointer;
}

.btn-confirm-delete {
  background: var(--danger-color);
  color: white;
  border: none;
  padding: 10px 22px;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
}

.empty-state {
  text-align: center;
  padding: 60px 20px;
  background: var(--bg-card);
  border-radius: 16px;
  border: 2px dashed var(--border-color);
}

.empty-icon { font-size: 3rem; margin-bottom: 10px; }

/* Calificación por Estrellas */
.star-buttons {
  display: flex;
  gap: 8px;
  margin: 10px 0;
}

.star-btn {
  background: #0f172a;
  border: 1px solid var(--border-color);
  color: #475569;
  font-size: 1.5rem;
  padding: 6px 14px;
  border-radius: 8px;
  cursor: pointer;
}

.star-btn.active {
  color: var(--accent-gold);
  border-color: var(--accent-gold);
}

.rating-label-selected {
  font-size: 0.85rem;
  color: var(--accent-gold);
  font-weight: bold;
}

.help-text {
  font-size: 0.85rem;
  color: var(--text-muted);
  margin-bottom: 14px;
}

.text-center { text-align: center; }
.warning-icon { font-size: 2.5rem; margin-bottom: 10px; }

@media (max-width: 768px) {
  .app-container { padding: 12px; }
  .barber-header { flex-direction: column; align-items: stretch; gap: 14px; text-align: center; }
  .brand-section { justify-content: center; }
  .form-grid { grid-template-columns: 1fr; }
  .controls-bar { flex-direction: column; align-items: stretch; }
  .sort-buttons { flex-wrap: wrap; justify-content: center; }
}

/* -------------------- AJUSTES SOLICITADOS -------------------- */

.main-content {
  width: 100%;
  flex: 1;
}

.payment-summary {
  width: 100%;
  background: linear-gradient(135deg, #1e293b 0%, #162033 100%);
  border: 1px solid var(--accent-gold);
  border-radius: 14px;
  padding: 18px 22px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
}

.payment-summary-title {
  display: flex;
  align-items: center;
  gap: 12px;
  min-width: 210px;
}

.payment-summary-title > span {
  font-size: 2rem;
}

.payment-summary-title h3 {
  color: var(--accent-gold);
  font-size: 1rem;
}

.payment-summary-title p {
  color: var(--text-muted);
  font-size: 0.78rem;
  margin-top: 3px;
}

.payment-summary-data {
  display: grid;
  grid-template-columns: repeat(4, minmax(130px, 1fr));
  gap: 12px;
  width: 100%;
}

.payment-summary-data div {
  background: #0f172a;
  border: 1px solid var(--border-color);
  border-radius: 9px;
  padding: 10px 12px;
}

.payment-summary-data span {
  display: block;
  color: var(--text-muted);
  font-size: 0.72rem;
  margin-bottom: 4px;
}

.payment-summary-data strong {
  font-size: 0.95rem;
  color: var(--text-main);
}

.saldo-text {
  color: var(--warning-color) !important;
}

.abonados-panel {
  border-color: var(--warning-color);
  background: rgba(245, 158, 11, 0.08);
}

.service-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  margin-top: 5px;
  padding-right: 70px;
}

.service-tag-card {
  margin-top: 0;
  background: rgba(245, 158, 11, 0.08);
  border: 1px solid rgba(245, 158, 11, 0.25);
  border-radius: 6px;
  padding: 3px 6px;
}

.card-abonado {
  border-top: 4px solid var(--warning-color);
}

.payment-status-pill.abonado {
  background: rgba(245, 158, 11, 0.2);
  color: var(--warning-color);
}

.abono-card-detail {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.abono-card-detail div {
  background: #0f172a;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  padding: 8px 10px;
}

.abono-card-detail span {
  display: block;
  color: var(--text-muted);
  font-size: 0.72rem;
}

.abono-card-detail strong {
  display: block;
  margin-top: 3px;
  color: var(--text-main);
}

.modal-register {
  max-width: 820px;
}

.modal-subtitle {
  color: var(--text-muted);
  font-size: 0.78rem;
  margin-top: 3px;
}

.services-selector {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 8px;
  background: #0f172a;
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 10px;
}

.service-option {
  display: flex;
  align-items: center;
  gap: 9px;
  padding: 10px;
  background: #1e293b;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  cursor: pointer;
  transition: 0.2s;
}

.service-option:hover,
.service-option.selected {
  border-color: var(--accent-gold);
  background: rgba(245, 158, 11, 0.1);
}

.service-option input {
  accent-color: var(--accent-gold);
}

.service-option-text {
=======
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
>>>>>>> c478cc73262e5a7342e9cf5bffc7c4942483dd10
  display: flex;
  flex-direction: column;
  gap: 2px;
}

<<<<<<< HEAD
.service-option-text strong {
  color: var(--text-main);
  font-size: 0.82rem;
}

.service-option-text span {
  color: var(--accent-gold);
  font-size: 0.75rem;
}

.form-total-box,
.grand-total-box {
  margin-top: 8px;
  padding: 11px 13px;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.form-total-box {
  background: rgba(16, 185, 129, 0.08);
  border: 1px solid rgba(16, 185, 129, 0.3);
}

.grand-total-box {
  background: #0f172a;
  border: 1px solid var(--accent-gold);
}

.form-total-box strong {
  color: var(--success-color);
}

.grand-total-box strong {
  color: var(--accent-gold);
  font-size: 1.15rem;
}

.abono-form-box {
  background: rgba(245, 158, 11, 0.08);
  border: 1px solid var(--accent-gold);
  padding: 12px;
  border-radius: 10px;
}

.abono-resumen {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 8px;
  margin-top: 10px;
}

.abono-resumen div {
  background: #0f172a;
  border-radius: 7px;
  padding: 8px;
}

.abono-resumen span {
  display: block;
  color: var(--text-muted);
  font-size: 0.72rem;
}

.abono-resumen strong {
  display: block;
  margin-top: 2px;
}

.form-help {
  color: var(--text-muted);
  font-size: 0.7rem;
}

@media (max-width: 900px) {
  .payment-summary {
    flex-direction: column;
    align-items: stretch;
  }

  .payment-summary-data {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 600px) {
  .services-selector {
    grid-template-columns: 1fr;
  }

  .payment-summary-data,
  .abono-resumen {
    grid-template-columns: 1fr;
  }

  .service-tags {
    padding-right: 0;
  }
}

</style>
=======
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
>>>>>>> c478cc73262e5a7342e9cf5bffc7c4942483dd10
