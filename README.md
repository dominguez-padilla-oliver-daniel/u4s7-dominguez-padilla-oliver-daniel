# U4S7 — Pulido funcional y UX
Dominguez Padilla Oliver Daniel · Matrícula 25311067 · Grupo TIDSM 4-1

| Recurso | Enlace |
|---------|--------|
| Producción | https://u4s7-dominguez-padilla-oliver-daniel.onrender.com |
| Repositorio | https://github.com/dominguez-padilla-oliver-daniel/u4s7-dominguez-padilla-oliver-daniel |

## Stack

| Capa | Tecnología |
|------|-----------|
| Backend | Django (Python), arquitectura MTV |
| Base de datos | MySQL gestionado en Aiven |
| Frontend | Plantillas Django + Tailwind CSS (Play CDN) |
| Servidor de aplicación | Gunicorn + WhiteNoise (estáticos) |
| Despliegue | Render (Web Service, despliegue continuo desde `main`) |
| Configuración | Variables de entorno con `python-decouple` |

## Funcionalidades

- **CRUD de servicios** con `ModelForm` y validación en servidor.
- **Autenticación**: registro, inicio y cierre de sesión con mensajes flash.
- **Aislamiento por propietario**: cada usuario solo ve y modifica sus propios servicios.
- **Paginación** de 5 registros por página.
- **Búsqueda por título** (`?q=`), insensible a mayúsculas.
- **Estados vacíos** diferenciados: sin registros y sin coincidencias de búsqueda.

## Semana 7: decisiones técnicas

### Listado con búsqueda y paginación (`core/views.py`)

El queryset se construye en este orden: **filtro por propietario → filtro de búsqueda → paginación**. Así las páginas se calculan sobre los resultados ya filtrados y nunca se exponen registros ajenos.

| Decisión | Justificación |
|----------|---------------|
| `order_by('-creado_en', '-id')` | `Paginator` exige un orden determinista. `-id` desempata registros creados en el mismo instante y evita duplicados u omisiones entre páginas |
| `titulo__icontains` | Búsqueda simple (`LIKE '%q%'`) suficiente para el volumen actual |
| `paginator.get_page()` | Tolera números de página inválidos: devuelve la primera o la última página en lugar de lanzar error |
| Formulario de búsqueda con `GET` | El estado queda en la URL: se puede recargar, compartir y navegar con el historial |
| `q` + `page` en los enlaces de paginación (`\|urlencode`) | Conserva la búsqueda al cambiar de página y escapa caracteres especiales |
| `{% empty %}` con mensaje condicional | Distingue "aún no hay datos" de "la búsqueda no encontró nada" |

## UI review: accesibilidad y responsividad

Se revisó la aplicación con la checklist del curso (6 puntos) sobre todas las plantillas y sobre la URL de producción.

| # | Criterio | Resultado |
|---|----------|-----------|
| 1 | `aria-label` en controles solo-ícono | Sin hallazgos: todos los botones y enlaces tienen texto visible; el campo de búsqueda ya cuenta con `aria-label` |
| 2 | Contraste de color | Hallazgo corregido |
| 3 | Navegación completa con teclado | Verificada |
| 4 | Responsividad a ~375 px | Hallazgo corregido |
| 5 | Áreas táctiles de ~44 px | Hallazgo corregido |
| 6 | Estado vacío | Sin hallazgos |

### Hallazgos y correcciones

1. **Contraste insuficiente en formularios** (`base.html`). El borde de los campos (`#94a3b8`, ≈ 2.6:1) y el placeholder (≈ 2.5:1) no alcanzaban el mínimo de WCAG AA. Se cambiaron a `#64748b` (≈ 4.8:1).
2. **Áreas táctiles pequeñas** (`servicios.html`, `base.html`, formularios). Los enlaces «Editar», «Eliminar» y «Cancelar» eran texto en línea de ≈ 20 px de alto y quedaban a 12 px entre sí, con riesgo de activar «Eliminar» por error. Se les asignó `min-h-[44px]` y padding.
3. **Falta de viewport en `inicio.html`**. La plantilla no heredaba de `base.html` ni declaraba `<meta name="viewport">`, por lo que en móvil se renderizaba como página de escritorio reducida. Se corrigió para heredar del molde base.

## Despliegue

El Web Service de Render ejecuta en cada deploy:

```bash
# Build Command
pip install -r requirements.txt && python manage.py collectstatic --noinput && python manage.py migrate

# Start Command
gunicorn miproyecto.wsgi
```

Variables de entorno requeridas (sus valores no se versionan): `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`, `SECRET_KEY`, `DEBUG`, `ALLOWED_HOSTS`, `CSRF_TRUSTED_ORIGINS`.

## Ejecución local

```bash
python -m venv venv
venv\Scripts\activate            # Windows (PowerShell)
pip install -r requirements.txt
# crear un archivo .env con las variables listadas arriba
python manage.py migrate
python manage.py runserver
```

## Estructura relevante

```
core/
├── models.py          # Servicio (propietario, titulo, precio, creado_en)
├── forms.py           # ServicioForm (ModelForm)
├── views.py           # CRUD, autenticación, búsqueda y paginación
└── templates/core/
    ├── base.html      # Molde: navbar, mensajes flash, estilos de formularios
    ├── servicios.html # Listado con buscador, tabla y paginación
    └── ...            # servicio_form, servicio_confirm_delete, login, registro
```
