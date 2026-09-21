<template>
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
        <button @click="abrirModalCatalogo" class="btn-secondary-header">🧾 Catálogo</button>
        <button @click="abrirModalCaja" class="btn-secondary-header">💰 Cerrar caja</button>
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

    <!-- COMISIONES DEL DÍA -->
    <section class="daily-panels">
      <div class="daily-panel commission-panel">
        <div class="panel-header">
          <span class="panel-icon">💼</span>
          <div>
            <h3>Comisiones de hoy</h3>
            <p>Porcentaje correspondiente a cada barbero</p>
          </div>
        </div>
        <div class="commission-grid">
          <div v-for="barbero in listaBarberos" :key="barbero.nombre" class="commission-card">
            <span class="commission-name">💈 {{ barbero.nombre }}</span>
            <span class="commission-percent">{{ barbero.comision }}%</span>
            <strong>${{ calcularComisionBarbero(barbero.nombre).toLocaleString('es-CO') }}</strong>
          </div>
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
              {{ servicio.estado }}
            </span>
          </div>

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

          <div v-if="servicio.fotoAntes || servicio.fotoDespues" class="card-photos">
            <div v-if="servicio.fotoAntes">
              <span>Antes</span>
              <img :src="servicio.fotoAntes" alt="Foto antes del servicio" />
            </div>
            <div v-if="servicio.fotoDespues">
              <span>Después</span>
              <img :src="servicio.fotoDespues" alt="Foto después del servicio" />
            </div>
          </div>

          <div class="card-actions">
            <span
              v-if="servicio.estado === 'Abonado' && calcularSaldoRegistro(servicio) > 0"
              class="servicio-abono-activo"
            >
              💰 Saldo pendiente: ${{ calcularSaldoRegistro(servicio).toLocaleString('es-CO') }}
            </span>

            <span v-else-if="servicioYaFinalizo(servicio)" class="servicio-finalizado">
              🔒 Servicio finalizado
            </span>

            <button
              @click="editarServicio(servicio)"
              :disabled="servicioYaFinalizo(servicio)"
              :class="['btn-action-edit', { 'btn-bloqueado': servicioYaFinalizo(servicio) }]"
            >
              ✏️ Editar
            </button>

            <button
              @click="confirmarEliminacion(servicio.id)"
              :disabled="servicioYaFinalizo(servicio)"
              :class="['btn-action-delete', { 'btn-bloqueado': servicioYaFinalizo(servicio) }]"
            >
              🗑️ Eliminar
            </button>
          </div>
        </div>
      </div>
    </main>

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
          🎉 ¡Cliente frecuente, aplica 10% de descuento!
          <span class="loyalty-count">{{ conteoClienteActual }} servicios registrados.</span>
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

              <div class="services-dropdown">
                <button
                  type="button"
                  @click="mostrarSelectorServicios = !mostrarSelectorServicios"
                  class="services-dropdown-button"
                >
                  <span>
                    ✂️
                    {{ form.serviciosSeleccionados.length > 0
                      ? form.serviciosSeleccionados.length + ' servicio(s) seleccionado(s)'
                      : 'Seleccione los servicios...' }}
                  </span>
                  <span>{{ mostrarSelectorServicios ? '▲' : '▼' }}</span>
                </button>

                <div v-show="mostrarSelectorServicios" class="services-dropdown-menu">
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
                      :disabled="!puedeSeleccionarServicio(item.nombre)"
                    />
                    <span class="service-option-text">
                      <strong>{{ item.nombre }}</strong>
                      <span>${{ item.precio.toLocaleString('es-CO') }}</span>
                    </span>
                    <span
                      v-if="form.serviciosSeleccionados.includes(item.nombre)"
                      class="service-check"
                    >
                      ✓
                    </span>
                  </label>
                </div>
              </div>

              <span class="form-help servicios-ayuda">
                Seleccione un solo corte. Puede agregar servicios complementarios como cejas o diseño de línea.
              </span>

              <span v-if="errores.servicios" class="error-msg">{{ errores.servicios }}</span>

              <div class="form-total-box">
                <span>Total de servicios:</span>
                <strong>${{ calcularTotalFormulario().toLocaleString('es-CO') }}</strong>
              </div>

              <div v-if="descuentoFidelidadActivo" class="discount-box">
                <span>🎁 Descuento fidelidad (10%):</span>
                <strong>-${{ calcularDescuentoFidelidad().toLocaleString('es-CO') }}</strong>
              </div>
            </div>

            <!-- BARBERO -->
            <div class="form-group">
              <label>Seleccione el barbero:</label>
              <select v-model="form.barbero" :class="{ 'input-error': errores.barbero }">
                <option value="">Seleccione el barbero...</option>
                <option v-for="b in listaBarberos" :key="b.nombre" :value="b.nombre">{{ b.nombre }}</option>
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
                <option value="Pendiente">⏳ Pendiente</option>
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

            <!-- FOTOS ANTES Y DESPUÉS -->
            <div class="form-group full-width photos-form-box">
              <label>Fotos del servicio (máximo 1 antes y 1 después):</label>
              <div class="photos-grid">
                <div class="photo-upload-box">
                  <span>📷 Foto antes</span>
                  <input type="file" accept="image/*" @change="cargarFotoAntes" />
                  <img v-if="form.fotoAntes" :src="form.fotoAntes" class="photo-preview" />
                </div>
                <div class="photo-upload-box">
                  <span>📷 Foto después</span>
                  <input type="file" accept="image/*" @change="cargarFotoDespues" />
                  <img v-if="form.fotoDespues" :src="form.fotoDespues" class="photo-preview" />
                </div>
              </div>
              <span class="form-help">Las fotos se guardan en base64. Para no llenar localStorage, cada foto debe pesar máximo 800 KB.</span>
              <span v-if="errores.fotos" class="error-msg">{{ errores.fotos }}</span>
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
            <button
              type="submit"
              class="btn-save"
              :disabled="registrandoCliente"
            >
              <span v-if="registrandoCliente" class="spinner"></span>
              <span v-if="registrandoCliente">Registrando...</span>
              <span v-else>{{ editandoId ? 'Actualizar Registro' : 'Guardar Servicio' }}</span>
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- MODAL CATÁLOGO -->
    <div v-show="mostrarModalCatalogo" class="modal-backdrop">
      <div class="modal-box modal-catalogo">
        <div class="modal-header">
          <div>
            <h2>🧾 Catálogo de servicios</h2>
            <p class="modal-subtitle">Don Ramiro puede crear, editar y eliminar servicios.</p>
          </div>
          <button @click="cerrarModalCatalogo" class="btn-close-modal">✖</button>
        </div>

        <div class="catalog-form">
          <input type="text" v-model="nuevoServicio.nombre" placeholder="Nombre del servicio" />
          <input type="text" inputmode="numeric" v-model="nuevoServicio.precio" placeholder="Precio base sugerido" />
          <select v-model="nuevoServicio.tipo">
            <option value="corte">Corte</option>
            <option value="corte-barba">Corte + barba</option>
            <option value="barba">Barba</option>
            <option value="complementario">Complementario</option>
          </select>
          <button @click="guardarServicioCatalogo" class="btn-save">{{ servicioCatalogoEditando ? 'Actualizar' : 'Agregar' }}</button>
        </div>
        <span v-if="errores.catalogo" class="error-msg">{{ errores.catalogo }}</span>

        <div class="catalog-list">
          <div v-for="item in catalogoServicios" :key="item.nombre" class="catalog-item">
            <div>
              <strong>{{ item.nombre }}</strong>
              <span>${{ Number(item.precio).toLocaleString('es-CO') }}</span>
            </div>
            <div class="catalog-actions">
              <button @click="editarServicioCatalogo(item)" class="btn-catalog-edit">✏️</button>
              <button @click="eliminarServicioCatalogo(item.nombre)" class="btn-catalog-delete">🗑️</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- MODAL CIERRE DE CAJA -->
    <div v-show="mostrarModalCaja" class="modal-backdrop">
      <div class="modal-box modal-caja">
        <div class="modal-header">
          <div>
            <h2>💰 Cierre de caja diario</h2>
            <p class="modal-subtitle">Resumen del {{ obtenerFechaHoyLocal() }}</p>
          </div>
          <button @click="cerrarModalCaja" class="btn-close-modal">✖</button>
        </div>

        <div class="cash-summary-grid">
          <div><span>💵 Total en efectivo</span><strong>${{ calcularCajaDelDia().efectivo.toLocaleString('es-CO') }}</strong></div>
          <div><span>📱 Total en transferencia</span><strong>${{ calcularCajaDelDia().transferencia.toLocaleString('es-CO') }}</strong></div>
          <div><span>💳 Total en tarjeta</span><strong>${{ calcularCajaDelDia().tarjeta.toLocaleString('es-CO') }}</strong></div>
          <div><span>⚠️ Pendiente por cobrar</span><strong>${{ calcularCajaDelDia().pendiente.toLocaleString('es-CO') }}</strong></div>
        </div>

        <div class="commission-summary">
          <h3>Comisiones del día</h3>
          <div v-for="barbero in listaBarberos" :key="barbero.nombre" class="commission-summary-row">
            <span>{{ barbero.nombre }} ({{ barbero.comision }}%)</span>
            <strong>${{ calcularComisionBarbero(barbero.nombre).toLocaleString('es-CO') }}</strong>
          </div>
        </div>

        <p class="help-text">Al archivar, los servicios de hoy dejarán de aparecer en la vista principal, pero seguirán guardados en localStorage.</p>

        <div class="modal-actions-right">
          <button @click="cerrarModalCaja" class="btn-cancel">Cancelar</button>
          <button @click="cerrarCajaYArchivar" class="btn-save">📦 Cerrar caja y archivar</button>
        </div>
      </div>
    </div>

    <!-- MODAL ELIMINAR -->
    <div v-show="mostrarModalEliminar" class="modal-backdrop">
      <div class="modal-box modal-small text-center">
        <div class="warning-icon">⚠️</div>
        <h3>Confirmar Eliminación</h3>
        <p>¿Seguro que deseas eliminar este registro de servicio? Esta acción no se puede deshacer.</p>

        <div class="modal-actions-center">
          <button @click="mostrarModalEliminar = false" class="btn-cancel">Cancelar</button>
          <button @click="ejecutarEliminacion" class="btn-confirm-delete">Sí, Eliminar</button>
        </div>
      </div>
    </div>

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
const catalogoServicios = useLocalStorage('don_ramiro_catalogo_servicios', [
  { nombre: 'Corte clásico con tijera', precio: 18000, tipo: 'corte' },
  { nombre: 'Low Fade', precio: 25000, tipo: 'corte' },
  { nombre: 'Mid Fade', precio: 25000, tipo: 'corte' },
  { nombre: 'High Fade', precio: 27000, tipo: 'corte' },
  { nombre: 'Taper Fade', precio: 25000, tipo: 'corte' },
  { nombre: 'Corte Buzz Cut', precio: 22000, tipo: 'corte' },
  { nombre: 'Corte + Barba', precio: 35000, tipo: 'corte-barba' },
  { nombre: 'Arreglo y perfilado de barba', precio: 15000, tipo: 'barba' },
  { nombre: 'Perfilado de cejas', precio: 7000, tipo: 'complementario' },
  { nombre: 'Diseño de línea', precio: 5000, tipo: 'complementario' }
])
const cierresCaja = useLocalStorage('don_ramiro_cierres_caja', [])

// Barberos y porcentaje de comisión
const listaBarberos = [
  { nombre: 'Don Ramiro', comision: 50 },
  { nombre: 'Empleado 1', comision: 40 },
  { nombre: 'Empleado 2', comision: 40 }
]

// Estados de interfaz
const mostrarModal = ref(false)
const editandoId = ref(null)
const mostrarModalEliminar = ref(false)
const idAEliminar = ref(null)
const mostrarModalCalificar = ref(false)
const calificandoId = ref(null)
const registrandoCliente = ref(false)

// Reloj reactivo: actualiza la hora cada segundo para que las tarjetas
// se bloqueen automáticamente justo cuando termina la hora de atención.
const relojActual = ref(Date.now())
setInterval(() => {
  relojActual.value = Date.now()
}, 1000)
const mostrarSelectorServicios = ref(false)
const mostrarModalCaja = ref(false)
const mostrarModalCatalogo = ref(false)
const servicioCatalogoEditando = ref('')

// Búsqueda y orden
const busquedaCliente = ref('')
const criterioOrden = ref('fecha-desc')
const alertaFrecuenteVisible = ref(false)
const conteoClienteActual = ref(0)
const descuentoFidelidadActivo = ref(false)

// Catálogo editable
const nuevoServicio = ref({ nombre: '', precio: '', tipo: 'complementario' })

// Formulario principal
function obtenerFormVacio() {
  return {
    cliente: '',
    serviciosSeleccionados: [],
    barbero: '',
    fecha: '',
    hora: '',
    propina: '',
    metodoPago: '',
    estado: '',
    abono: '',
    fotoAntes: '',
    fotoDespues: ''
  }
}

const form = ref(obtenerFormVacio())

// Formulario de calificación separado del registro
const formCalificacion = ref({
  calificacion: 5,
  observaciones: ''
})

// Errores manejados por JavaScript
const errores = ref({})

// -------------------- FECHA Y HORA --------------------
function obtenerFechaHoyLocal() {
  const hoy = new Date(relojActual.value)
  const anio = hoy.getFullYear()
  const mes = String(hoy.getMonth() + 1).padStart(2, '0')
  const dia = String(hoy.getDate()).padStart(2, '0')
  return `${anio}-${mes}-${dia}`
}

function obtenerHoraActualLocal() {
  const hoy = new Date(relojActual.value)
  return `${String(hoy.getHours()).padStart(2, '0')}:${String(hoy.getMinutes()).padStart(2, '0')}`
}

function fechaEsAnteriorAHoy(fecha) {
  return fecha < obtenerFechaHoyLocal()
}

// -------------------- SERVICIOS --------------------
function obtenerTipoServicio(nombreServicio) {
  const item = catalogoServicios.value.find(s => s.nombre === nombreServicio)
  if (item && item.tipo) return item.tipo

  const nombre = String(nombreServicio || '').toLowerCase()
  if (nombre.includes('corte') || nombre.includes('fade') || nombre.includes('buzz') || nombre.includes('taper')) return 'corte'
  if (nombre.includes('barba')) return nombre.includes('corte') ? 'corte-barba' : 'barba'
  return 'complementario'
}

function esCorte(nombreServicio) {
  const tipo = obtenerTipoServicio(nombreServicio)
  return tipo === 'corte' || tipo === 'corte-barba'
}

function esServicioConBarba(nombreServicio) {
  return obtenerTipoServicio(nombreServicio) === 'corte-barba'
}

function esArregloBarba(nombreServicio) {
  return obtenerTipoServicio(nombreServicio) === 'barba'
}

function puedeSeleccionarServicio(nombreServicio) {
  const seleccionados = form.value.serviciosSeleccionados

  if (seleccionados.includes(nombreServicio)) return true

  if (esCorte(nombreServicio)) {
    for (let i = 0; i < seleccionados.length; i++) {
      if (esCorte(seleccionados[i])) return false
    }
  }

  if (esServicioConBarba(nombreServicio) && seleccionados.includes('Arreglo y perfilado de barba')) return false
  if (esArregloBarba(nombreServicio) && seleccionados.includes('Corte + Barba')) return false

  return true
}

function obtenerServiciosDelRegistro(servicio) {
  if (servicio.servicios && servicio.servicios.length > 0) return servicio.servicios
  return servicio.tipoServicio ? [servicio.tipoServicio] : []
}

function calcularTotalBaseServicios(serviciosSeleccionados) {
  let total = 0

  for (let i = 0; i < serviciosSeleccionados.length; i++) {
    const item = catalogoServicios.value.find(s => s.nombre === serviciosSeleccionados[i])
    if (item) total += Number(item.precio || 0)
  }

  return total
}

function calcularDescuentoFidelidad() {
  if (!descuentoFidelidadActivo.value) return 0
  return Math.round(calcularTotalBaseServicios(form.value.serviciosSeleccionados) * 0.10)
}

function calcularTotalFormulario() {
  const base = calcularTotalBaseServicios(form.value.serviciosSeleccionados)
  return base - calcularDescuentoFidelidad()
}

function calcularTotalConPropina() {
  return calcularTotalFormulario() + Number(form.value.propina || 0)
}

function calcularSaldoFormulario() {
  const total = calcularTotalConPropina()
  const abono = Number(form.value.abono || 0)
  return Math.max(0, total - abono)
}

function calcularSaldoRegistro(servicio) {
  const total = Number(servicio.totalConPropina ?? (Number(servicio.precio || 0) + Number(servicio.propina || 0)))
  return Math.max(0, total - Number(servicio.abono || 0))
}

// -------------------- HISTORIAL Y ESTADÍSTICAS --------------------
function contarServiciosCliente(nombre) {
  const nombreNormalizado = String(nombre || '').trim().toLowerCase()
  if (!nombreNormalizado) return 0

  return servicios.value.filter(s => String(s.cliente || '').trim().toLowerCase() === nombreNormalizado).length
}

function calcularGastoTotalCliente(nombre) {
  const nombreNormalizado = String(nombre || '').trim().toLowerCase()
  return servicios.value
    .filter(s => String(s.cliente || '').trim().toLowerCase() === nombreNormalizado)
    .reduce((total, s) => total + Number(s.totalConPropina ?? (Number(s.precio || 0) + Number(s.propina || 0))), 0)
}

function obtenerServiciosFiltrados() {
  let lista = servicios.value.filter(s => !s.archivado)

  if (busquedaCliente.value.trim()) {
    const busqueda = busquedaCliente.value.trim().toLowerCase()
    lista = lista.filter(s => String(s.cliente || '').toLowerCase().includes(busqueda))
  }

  if (criterioOrden.value === 'precio-desc') {
    lista.sort((a, b) => Number(b.totalConPropina ?? b.precio ?? 0) - Number(a.totalConPropina ?? a.precio ?? 0))
  } else if (criterioOrden.value === 'calificacion-desc') {
    lista.sort((a, b) => Number(b.calificacion || 0) - Number(a.calificacion || 0))
  } else {
    lista.sort((a, b) => `${b.fecha || ''} ${b.hora || ''}`.localeCompare(`${a.fecha || ''} ${a.hora || ''}`))
  }

  return lista
}

function calcularIngresosTotales() {
  return servicios.value
    .filter(s => !s.archivado)
    .reduce((total, s) => total + Number(s.abono || 0), 0)
}

function contarAbonados() {
  return servicios.value.filter(s => !s.archivado && s.estado === 'Abonado' && calcularSaldoRegistro(s) > 0).length
}

function calcularPromedioCalificacion() {
  const calificadas = servicios.value.filter(s => !s.archivado && Number(s.calificacion || 0) > 0)
  if (calificadas.length === 0) return '0.0'
  const total = calificadas.reduce((sum, s) => sum + Number(s.calificacion), 0)
  return (total / calificadas.length).toFixed(1)
}

function obtenerBarberoDestacado() {
  const conteo = {}
  servicios.value.filter(s => !s.archivado).forEach(s => {
    if (!conteo[s.barbero]) conteo[s.barbero] = 0
    conteo[s.barbero]++
  })

  let nombre = 'Sin registros'
  let mayor = 0
  Object.keys(conteo).forEach(nombreBarbero => {
    if (conteo[nombreBarbero] > mayor) {
      mayor = conteo[nombreBarbero]
      nombre = nombreBarbero
    }
  })
  return nombre
}

function obtenerUltimoAbono() {
  const lista = servicios.value.filter(s => Number(s.abono || 0) > 0)
  if (lista.length === 0) return null
  const ultimo = lista[lista.length - 1]
  return {
    cliente: ultimo.cliente,
    abono: Number(ultimo.abono || 0),
    total: Number(ultimo.totalConPropina ?? (Number(ultimo.precio || 0) + Number(ultimo.propina || 0))),
    saldo: calcularSaldoRegistro(ultimo)
  }
}

function obtenerListaAbonados() {
  return servicios.value
    .filter(s => !s.archivado && calcularSaldoRegistro(s) > 0)
    .map(s => ({ cliente: s.cliente, saldo: calcularSaldoRegistro(s) }))
}

function obtenerNombreTurno(hora) {
  if (!hora) return ''
  const h = Number(String(hora).split(':')[0])
  if (h < 12) return 'Mañana'
  if (h < 18) return 'Tarde'
  return 'Noche'
}

function obtenerClaseTurno(hora) {
  if (!hora) return ''
  const h = Number(String(hora).split(':')[0])
  if (h < 12) return 'shift-morning'
  if (h < 18) return 'shift-afternoon'
  return 'shift-night'
}

// -------------------- FIDELIDAD --------------------
function verificarClienteFrecuente() {
  if (editandoId.value) return

  const nombre = form.value.cliente.trim()
  if (nombre.length < 2) {
    conteoClienteActual.value = 0
    descuentoFidelidadActivo.value = false
    alertaFrecuenteVisible.value = false
    return
  }

  const conteo = contarServiciosCliente(nombre)
  conteoClienteActual.value = conteo
  descuentoFidelidadActivo.value = conteo >= 5
  alertaFrecuenteVisible.value = conteo >= 5
}

// -------------------- ABRIR / EDITAR --------------------
function abrirNuevoModal() {
  registrandoCliente.value = false
  form.value = obtenerFormVacio()
  editandoId.value = null
  errores.value = {}
  alertaFrecuenteVisible.value = false
  descuentoFidelidadActivo.value = false
  conteoClienteActual.value = 0
  mostrarSelectorServicios.value = false
  mostrarModal.value = true
}

function editarServicio(servicio) {
  if (servicioYaFinalizo(servicio)) return

  registrandoCliente.value = false
  form.value = {
    cliente: servicio.cliente,
    serviciosSeleccionados: obtenerServiciosDelRegistro(servicio),
    barbero: servicio.barbero,
    fecha: servicio.fecha,
    hora: servicio.hora,
    propina: servicio.propina || '',
    metodoPago: servicio.metodoPago || '',
    estado: servicio.estado || '',
    abono: servicio.abono || '',
    fotoAntes: servicio.fotoAntes || '',
    fotoDespues: servicio.fotoDespues || ''
  }

  editandoId.value = servicio.id
  errores.value = {}
  mostrarSelectorServicios.value = false
  descuentoFidelidadActivo.value = Boolean(servicio.descuentoFidelidad)
  alertaFrecuenteVisible.value = false
  mostrarModal.value = true
}

function cerrarModal() {
  registrandoCliente.value = false
  mostrarModal.value = false
  errores.value = {}
  mostrarSelectorServicios.value = false
  alertaFrecuenteVisible.value = false
  descuentoFidelidadActivo.value = false
  conteoClienteActual.value = 0
  form.value = obtenerFormVacio()
}

// -------------------- VALIDACIONES --------------------
function validarFechaYHora(err) {
  if (!form.value.fecha) {
    err.fecha = 'Seleccione una fecha.'
  }

  if (!form.value.hora) {
    err.hora = 'Seleccione una hora.'
    return
  }

  // Cuando estamos editando un servicio existente, permitimos conservar
  // su fecha y hora aunque ya hayan pasado. Esto es necesario para poder
  // actualizar un abono a "Pagado" y registrar el pago pendiente.
  let esFechaHoraHistorica = false

  if (editandoId.value) {
    const servicioOriginal = servicios.value.find(s => s.id === editandoId.value)

    if (
      servicioOriginal &&
      servicioOriginal.fecha === form.value.fecha &&
      servicioOriginal.hora === form.value.hora
    ) {
      esFechaHoraHistorica = true
    }
  }

  // Para un registro nuevo sí se mantienen las validaciones normales.
  // Para una edición, solo se permite una fecha/hora vencida si es la
  // misma fecha/hora que ya tenía el servicio.
  if (!esFechaHoraHistorica) {
    if (fechaEsAnteriorAHoy(form.value.fecha)) {
      err.fecha = 'No puede seleccionar una fecha que ya pasó.'
    }

    if (form.value.fecha === obtenerFechaHoyLocal() && form.value.hora < obtenerHoraActualLocal()) {
      err.hora = 'La hora seleccionada ya pasó.'
    }
  }

  if (form.value.hora < '08:00' || form.value.hora > '20:00') {
    err.hora = 'El horario de atención es de 8:00 a. m. a 8:00 p. m.'
  }
}

function validarFormulario() {
  const err = {}

  if (!form.value.cliente || form.value.cliente.trim().length < 2) err.cliente = 'Debe ingresar el nombre del cliente.'
  if (!form.value.serviciosSeleccionados || form.value.serviciosSeleccionados.length === 0) err.servicios = 'Debe seleccionar al menos un servicio.'
  if (!form.value.barbero) err.barbero = 'Seleccione el barbero que atendió.'

  validarFechaYHora(err)

  const propina = Number(form.value.propina || 0)
  if (form.value.propina !== '' && (isNaN(propina) || propina < 0)) err.propina = 'La propina debe ser un valor válido de 0 o mayor.'

  if (!form.value.metodoPago) err.metodoPago = 'Seleccione el método de pago.'
  if (!form.value.estado) err.estado = 'Seleccione el estado del pago.'

  const total = calcularTotalConPropina()
  if (total <= 0) err.servicios = 'Seleccione servicios para calcular el total.'

  if (form.value.estado === 'Abonado') {
    const abono = Number(form.value.abono || 0)
    if (!form.value.abono || isNaN(abono) || abono <= 0) err.abono = 'Ingrese cuánto abonó el cliente.'
    else if (abono >= total) err.abono = 'El abono debe ser menor que el total. Si paga todo, seleccione Pagado.'
  }

  if (form.value.estado === 'Pagado') {
    form.value.abono = String(total)
  }

  if (form.value.estado === 'Pendiente') {
    form.value.abono = '0'
  }

  errores.value = err
  return Object.keys(err).length === 0
}

// -------------------- FOTOS BASE64 --------------------
function leerFotoBase64(evento, campo) {
  const archivo = evento.target.files && evento.target.files[0]
  if (!archivo) return

  if (!archivo.type.startsWith('image/')) {
    errores.value = { ...errores.value, fotos: 'Seleccione un archivo de imagen.' }
    evento.target.value = ''
    return
  }

  if (archivo.size > 800 * 1024) {
    errores.value = { ...errores.value, fotos: 'Cada foto debe pesar máximo 800 KB para cuidar localStorage.' }
    evento.target.value = ''
    return
  }

  const lector = new FileReader()
  lector.onload = () => {
    form.value[campo] = lector.result
    errores.value = { ...errores.value, fotos: '' }
  }
  lector.readAsDataURL(archivo)
}

function cargarFotoAntes(evento) {
  leerFotoBase64(evento, 'fotoAntes')
}

function cargarFotoDespues(evento) {
  leerFotoBase64(evento, 'fotoDespues')
}

// -------------------- GUARDAR SERVICIO --------------------
function guardarServicio() {
  if (registrandoCliente.value) return
  if (!validarFormulario()) return

  registrandoCliente.value = true

  const total = calcularTotalFormulario()
  const propina = Number(form.value.propina || 0)
  const totalConPropina = total + propina
  const abono = form.value.estado === 'Abonado' ? Number(form.value.abono) : (form.value.estado === 'Pagado' ? totalConPropina : 0)
  const descuento = calcularDescuentoFidelidad()

  const datos = {
    cliente: form.value.cliente.trim(),
    servicios: [...form.value.serviciosSeleccionados],
    tipoServicio: form.value.serviciosSeleccionados[0] || '',
    barbero: form.value.barbero,
    fecha: form.value.fecha,
    hora: form.value.hora,
    precio: total,
    precioBase: calcularTotalBaseServicios(form.value.serviciosSeleccionados),
    descuento: descuento,
    descuentoFidelidad: descuento > 0,
    propina: propina,
    totalConPropina: totalConPropina,
    metodoPago: form.value.metodoPago,
    estado: form.value.estado,
    abono: abono,
    fotoAntes: form.value.fotoAntes || '',
    fotoDespues: form.value.fotoDespues || '',
    comisionPorcentaje: obtenerComisionPorcentaje(form.value.barbero)
  }

  setTimeout(() => {
    if (editandoId.value) {
      const idx = servicios.value.findIndex(s => s.id === editandoId.value)
      if (idx !== -1) servicios.value[idx] = { ...servicios.value[idx], ...datos }
    } else {
      servicios.value.push({
        id: Date.now(),
        ...datos,
        calificacion: 0,
        observaciones: '',
        archivado: false
      })
    }

    registrandoCliente.value = false
    cerrarModal()
  }, 1500)
}

// -------------------- BLOQUEO FINAL --------------------
function servicioYaFinalizo(servicio) {
  if (!servicio.fecha || !servicio.hora) {
    return false
  }

  // Solo se bloquea cuando el pago está completamente realizado.
  if (servicio.estado !== 'Pagado') {
    return false
  }

  // Debe tener calificación registrada.
  // Las observaciones son opcionales: no influyen en el bloqueo.
  const tieneCalificacion = Number(servicio.calificacion || 0) > 0

  if (!tieneCalificacion) {
    return false
  }

  // Convertir la fecha y hora del servicio en un momento exacto.
  const partesFecha = String(servicio.fecha).split('-')
  const partesHora = String(servicio.hora).split(':')

  if (partesFecha.length !== 3 || partesHora.length < 2) {
    return false
  }

  const anio = Number(partesFecha[0])
  const mes = Number(partesFecha[1]) - 1
  const dia = Number(partesFecha[2])
  const horas = Number(partesHora[0])
  const minutos = Number(partesHora[1])

  const fechaHoraServicio = new Date(
    anio,
    mes,
    dia,
    horas,
    minutos,
    0,
    0
  ).getTime()

  // El reloj se actualiza cada segundo.
  return relojActual.value >= fechaHoraServicio
}

function confirmarEliminacion(id) {
  const servicio = servicios.value.find(s => s.id === id)
  if (!servicio || servicioYaFinalizo(servicio)) return
  idAEliminar.value = id
  mostrarModalEliminar.value = true
}

function ejecutarEliminacion() {
  servicios.value = servicios.value.filter(s => s.id !== idAEliminar.value)
  mostrarModalEliminar.value = false
  idAEliminar.value = null
}

// -------------------- CALIFICACIÓN --------------------
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

// -------------------- CIERRE DE CAJA --------------------
function obtenerServiciosDelDia() {
  return servicios.value.filter(s => s.fecha === obtenerFechaHoyLocal() && !s.archivado)
}

function obtenerMontoCobrado(servicio) {
  if (servicio.estado === 'Pagado') return Number(servicio.totalConPropina ?? (Number(servicio.precio || 0) + Number(servicio.propina || 0)))
  if (servicio.estado === 'Abonado') return Number(servicio.abono || 0)
  return 0
}

function calcularCajaDelDia() {
  const resumen = { efectivo: 0, transferencia: 0, tarjeta: 0, pendiente: 0 }

  obtenerServiciosDelDia().forEach(servicio => {
    const cobrado = obtenerMontoCobrado(servicio)
    if (servicio.metodoPago === 'Efectivo') resumen.efectivo += cobrado
    if (servicio.metodoPago === 'Transferencia') resumen.transferencia += cobrado
    if (servicio.metodoPago === 'Tarjeta') resumen.tarjeta += cobrado
    resumen.pendiente += calcularSaldoRegistro(servicio)
  })

  return resumen
}

function obtenerComisionPorcentaje(nombre) {
  const barbero = listaBarberos.find(b => b.nombre === nombre)
  return barbero ? Number(barbero.comision) : 0
}

function calcularComisionBarbero(nombre) {
  return obtenerServiciosDelDia()
    .filter(s => s.barbero === nombre)
    .reduce((total, s) => {
      const porcentaje = Number(s.comisionPorcentaje ?? obtenerComisionPorcentaje(nombre))
      return total + (Number(s.precio || 0) * porcentaje / 100)
    }, 0)
}

function abrirModalCaja() {
  mostrarModalCaja.value = true
}

function cerrarModalCaja() {
  mostrarModalCaja.value = false
}

function cerrarCajaYArchivar() {
  const delDia = obtenerServiciosDelDia()
  const resumen = calcularCajaDelDia()

  if (delDia.length > 0) {
    delDia.forEach(servicio => {
      const idx = servicios.value.findIndex(s => s.id === servicio.id)
      if (idx !== -1) servicios.value[idx].archivado = true
    })
  }

  cierresCaja.value.push({
    id: Date.now(),
    fecha: obtenerFechaHoyLocal(),
    ...resumen,
    comisiones: listaBarberos.map(b => ({
      barbero: b.nombre,
      porcentaje: b.comision,
      valor: calcularComisionBarbero(b.nombre)
    }))
  })

  cerrarModalCaja()
}

// -------------------- CATÁLOGO EDITABLE --------------------
function abrirModalCatalogo() {
  servicioCatalogoEditando.value = ''
  nuevoServicio.value = { nombre: '', precio: '', tipo: 'complementario' }
  errores.value = {}
  mostrarModalCatalogo.value = true
}

function cerrarModalCatalogo() {
  mostrarModalCatalogo.value = false
  servicioCatalogoEditando.value = ''
  nuevoServicio.value = { nombre: '', precio: '', tipo: 'complementario' }
}

function editarServicioCatalogo(item) {
  servicioCatalogoEditando.value = item.nombre
  nuevoServicio.value = { nombre: item.nombre, precio: item.precio, tipo: item.tipo || 'complementario' }
}

function guardarServicioCatalogo() {
  const nombre = nuevoServicio.value.nombre.trim()
  const precio = Number(nuevoServicio.value.precio)

  if (!nombre) {
    errores.value = { catalogo: 'Ingrese el nombre del servicio.' }
    return
  }

  if (isNaN(precio) || precio <= 0) {
    errores.value = { catalogo: 'Ingrese un precio base mayor que 0.' }
    return
  }

  const existe = catalogoServicios.value.some(s => s.nombre.toLowerCase() === nombre.toLowerCase() && s.nombre !== servicioCatalogoEditando.value)
  if (existe) {
    errores.value = { catalogo: 'Ese servicio ya existe.' }
    return
  }

  if (servicioCatalogoEditando.value) {
    const idx = catalogoServicios.value.findIndex(s => s.nombre === servicioCatalogoEditando.value)
    if (idx !== -1) catalogoServicios.value[idx] = { nombre, precio, tipo: nuevoServicio.value.tipo }
  } else {
    catalogoServicios.value.push({ nombre, precio, tipo: nuevoServicio.value.tipo })
  }

  errores.value = {}
  servicioCatalogoEditando.value = ''
  nuevoServicio.value = { nombre: '', precio: '' }
}

function eliminarServicioCatalogo(nombre) {
  if (catalogoServicios.value.length <= 1) return
  catalogoServicios.value = catalogoServicios.value.filter(s => s.nombre !== nombre)
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
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

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

.btn-save:disabled {
  opacity: 0.75;
  cursor: not-allowed;
}

.spinner {
  display: inline-block;
  width: 15px;
  height: 15px;
  margin-right: 8px;
  border: 2px solid rgba(15, 23, 42, 0.35);
  border-top-color: #0f172a;
  border-radius: 50%;
  vertical-align: -3px;
  animation: girar-spinner 0.7s linear infinite;
}

@keyframes girar-spinner {
  to {
    transform: rotate(360deg);
  }
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


/* Fecha y hora: mismas cajas oscuras, controles más visibles */
.form-group input[type="date"],
.form-group input[type="time"] {
  background: #0f172a;
  color: #e2e8f0;
  color-scheme: dark;
  border: 1px solid var(--border-color);
  font-weight: 700;
  min-height: 44px;
}

.form-group input[type="date"]:focus,
.form-group input[type="time"]:focus {
  background: #0f172a;
  color: #f8fafc;
  border-color: var(--accent-gold);
}

.form-group input[type="date"]::-webkit-calendar-picker-indicator,
.form-group input[type="time"]::-webkit-calendar-picker-indicator {
  opacity: 1;
  cursor: pointer;
  width: 20px;
  height: 20px;
  filter: invert(1);
}

.servicios-ayuda {
  display: block;
  margin-top: 2px;
  color: #cbd5e1;
}


.servicio-abono-activo {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 8px;
  border-radius: 8px;
  background: rgba(245, 158, 11, 0.12);
  border: 1px solid rgba(245, 158, 11, 0.35);
  color: var(--warning-color);
  font-size: 0.78rem;
  font-weight: 800;
}

.servicio-finalizado {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 8px;
  border-radius: 8px;
  background: rgba(239, 68, 68, 0.12);
  border: 1px solid rgba(239, 68, 68, 0.35);
  color: #fca5a5;
  font-size: 0.78rem;
  font-weight: 800;
}

.btn-bloqueado {
  opacity: 0.45 !important;
  cursor: not-allowed !important;
  pointer-events: none;
}

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

.services-dropdown {
  position: relative;
  width: 100%;
}

.services-dropdown-button {
  width: 100%;
  min-height: 48px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding: 12px 14px;
  background: #0f172a;
  color: var(--text-main);
  border: 1px solid var(--border-color);
  border-radius: 9px;
  font-size: 0.88rem;
  font-weight: 700;
  cursor: pointer;
  text-align: left;
  transition: 0.2s;
}

.services-dropdown-button:hover {
  border-color: var(--accent-gold);
}

.services-dropdown-menu {
  margin-top: 7px;
  max-height: 300px;
  overflow-y: auto;
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 8px;
  background: #0f172a;
  border: 1px solid var(--accent-gold);
  border-radius: 10px;
  padding: 10px;
}

.service-option {
  display: flex;
  align-items: center;
  gap: 9px;
  min-height: 58px;
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

.service-option:has(input:disabled) {
  opacity: 0.5;
  cursor: not-allowed;
}

.service-option:has(input:disabled):hover {
  border-color: var(--border-color);
  background: #1e293b;
}

.service-option input {
  width: 17px;
  height: 17px;
  flex-shrink: 0;
  accent-color: var(--accent-gold);
}

.service-option-text {
  display: flex;
  flex-direction: column;
  gap: 2px;
  flex: 1;
}

.service-option-text strong {
  color: var(--text-main);
  font-size: 0.82rem;
}

.service-option-text span {
  color: var(--accent-gold);
  font-size: 0.75rem;
}

.service-check {
  color: var(--success-color);
  font-size: 1.1rem;
  font-weight: 900;
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
  .servicio-finalizado {
    grid-column: 1 / -1;
  }

  .services-dropdown-menu {
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


/* -------------------- NUEVAS FUNCIONES DEL SISTEMA -------------------- */
.header-actions {
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-end;
  gap: 8px;
}

.btn-secondary-header {
  background: #334155;
  color: var(--text-main);
  border: 1px solid var(--border-color);
  padding: 12px 16px;
  border-radius: 10px;
  font-weight: 800;
  cursor: pointer;
}

.btn-secondary-header:hover {
  border-color: var(--accent-gold);
  color: var(--accent-gold);
}

.daily-panels {
  width: 100%;
}

.daily-panel {
  background: var(--bg-card);
  border: 1px solid var(--border-color);
  border-radius: 14px;
  padding: 16px 20px;
}

.daily-panel .panel-header {
  margin-bottom: 12px;
}

.daily-panel .panel-header h3 {
  color: var(--accent-gold);
}

.daily-panel .panel-header p {
  color: var(--text-muted);
  font-size: 0.76rem;
}

.commission-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.commission-card {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 4px 10px;
  padding: 12px;
  background: #0f172a;
  border: 1px solid var(--border-color);
  border-radius: 10px;
}

.commission-name {
  color: var(--text-main);
  font-weight: 700;
}

.commission-percent {
  color: var(--accent-gold);
  font-weight: 800;
}

.commission-card strong {
  grid-column: 1 / -1;
  color: var(--success-color);
  font-size: 1.05rem;
}

.loyalty-count {
  display: block;
  color: var(--text-main);
  font-size: 0.75rem;
  margin-top: 3px;
}

.discount-box {
  margin-top: 8px;
  padding: 10px 13px;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  background: rgba(245, 158, 11, 0.10);
  border: 1px solid rgba(245, 158, 11, 0.35);
  color: var(--accent-gold);
}

.photos-form-box {
  background: rgba(15, 23, 42, 0.5);
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 12px;
}

.photos-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  margin-top: 8px;
}

.photo-upload-box {
  background: #0f172a;
  border: 1px dashed var(--border-color);
  border-radius: 9px;
  padding: 10px;
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.photo-upload-box > span {
  color: var(--text-main);
  font-weight: 700;
  font-size: 0.8rem;
}

.photo-upload-box input[type="file"] {
  padding: 7px;
  font-size: 0.75rem;
}

.photo-preview {
  width: 100%;
  height: 130px;
  object-fit: cover;
  border-radius: 8px;
  border: 1px solid var(--border-color);
}

.card-photos {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
  margin-bottom: 12px;
}

.card-photos div {
  background: #0f172a;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  padding: 5px;
}

.card-photos span {
  display: block;
  color: var(--text-muted);
  font-size: 0.7rem;
  margin-bottom: 4px;
}

.card-photos img {
  width: 100%;
  height: 100px;
  object-fit: cover;
  border-radius: 6px;
}

.modal-caja,
.modal-catalogo {
  max-width: 760px;
}

.cash-summary-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}

.cash-summary-grid div {
  background: #0f172a;
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 14px;
}

.cash-summary-grid span {
  display: block;
  color: var(--text-muted);
  font-size: 0.78rem;
  margin-bottom: 5px;
}

.cash-summary-grid strong {
  color: var(--success-color);
  font-size: 1.1rem;
}

.commission-summary {
  margin-top: 14px;
  background: #0f172a;
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 14px;
}

.commission-summary h3 {
  color: var(--accent-gold);
  font-size: 0.95rem;
  margin-bottom: 8px;
}

.commission-summary-row {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px solid var(--border-color);
  font-size: 0.84rem;
}

.commission-summary-row:last-child {
  border-bottom: none;
}

.commission-summary-row strong {
  color: var(--success-color);
}

.catalog-form {
  display: grid;
  grid-template-columns: 1.5fr 1fr 1fr auto;
  gap: 8px;
  margin-bottom: 8px;
}

.catalog-form input,
.catalog-form select {
  background: #0f172a;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  padding: 10px;
  color: var(--text-main);
}

.catalog-list {
  display: flex;
  flex-direction: column;
  gap: 7px;
  margin-top: 12px;
}

.catalog-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
  background: #0f172a;
  border: 1px solid var(--border-color);
  border-radius: 9px;
  padding: 10px 12px;
}

.catalog-item > div:first-child {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.catalog-item strong {
  color: var(--text-main);
}

.catalog-item span {
  color: var(--accent-gold);
  font-size: 0.78rem;
}

.catalog-actions {
  display: flex;
  gap: 6px;
}

.btn-catalog-edit,
.btn-catalog-delete {
  border: none;
  border-radius: 7px;
  padding: 7px 9px;
  cursor: pointer;
}

.btn-catalog-edit {
  background: #334155;
  color: var(--text-main);
}

.btn-catalog-delete {
  background: rgba(239, 68, 68, 0.18);
  color: var(--danger-color);
}

@media (max-width: 900px) {
  .commission-grid {
    grid-template-columns: 1fr;
  }

  .catalog-form {
    grid-template-columns: 1fr 1fr;
  }
}

@media (max-width: 600px) {
  .header-actions {
    justify-content: stretch;
  }

  .header-actions button {
    flex: 1;
  }

  .photos-grid,
  .cash-summary-grid,
  .catalog-form {
    grid-template-columns: 1fr;
  }

  .card-photos {
    grid-template-columns: 1fr;
  }
}


/* Iconos de calendario y reloj más visibles */
.form-group input[type="date"]::-webkit-calendar-picker-indicator,
.form-group input[type="time"]::-webkit-calendar-picker-indicator {
  opacity: 1;
  cursor: pointer;
  width: 22px;
  height: 22px;
  filter: brightness(0) invert(1);
}

.form-group input[type="date"],
.form-group input[type="time"] {
  color-scheme: dark;
}

</style>
