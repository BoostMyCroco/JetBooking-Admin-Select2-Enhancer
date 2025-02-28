# JetBooking-Admin-Select2-Enhancer

## Description

This script improves the user experience within the JetBooking admin popup by:
- Sorting `<select>` options alphabetically.
- Simulate the Select2 library for a modern, searchable dropdown.
- Ensuring the dropdown works seamlessly inside modals or popups.

## Features

- **Alphabetical Sorting**: Automatically sorts dropdown options in ascending order.
- **Select2 Simulation**: Provides a clean and searchable interface for dropdowns.
- **Popup Compatibility**: Ensures proper rendering within JetBooking modals.

## Installation

1. Copy the script into your WordPress installation.
2. Include the script in your theme's `functions.php` file or in a custom plugin.


## Usage

1. This script targets dropdowns within the JetBooking admin interface, specifically those with the class `.jet-abaf-details__field-apartment_id select`.
2. The dropdown will automatically enhance itself with alphabetical sorting like Select2 functionality whenever the modal or popup loads.

## Dependencies

jQuery: Ensure jQuery is loaded in your WordPress installation.

## Customization

If you need to modify the dropdown parent or other options:

Update the dropdownParent option in the script to match your modal structure.

## Screenshots

Before 

![image](https://github.com/user-attachments/assets/4429c7dd-4cb4-4bce-8c20-2d0afba1be68)

After

![image](https://github.com/user-attachments/assets/c6b6ae56-7200-45bf-a500-a2fe92c219c5)

## License
GNU General Public License v3.0

## Code

```php
<?php
add_action('admin_footer', function () {
    ?>
    <script>
        // Función para aplicar el scroll horizontal, ordenar el select y agregar el buscador
        const enhanceSelectWithScrollAndSearch = () => {
            const select = document.querySelector('.jet-abaf-details__field-apartment_id select');
            if (select && !select.hasAttribute('data-custom-select-initialized')) {
                // Crear un contenedor personalizado para el select
                const wrapper = document.createElement('div');
                wrapper.classList.add('custom-select-wrapper');
                wrapper.style.position = 'relative';
                wrapper.style.width = '100%';

                // Crear el campo de búsqueda
                const searchInput = document.createElement('input');
                searchInput.setAttribute('type', 'text');
                searchInput.setAttribute('placeholder', 'Buscar...');
                searchInput.style.padding = '8px';
                searchInput.style.marginBottom = '5px';
                searchInput.style.width = '100%';
                searchInput.style.boxSizing = 'border-box';

                // Crear un contenedor para el dropdown
                const dropdown = document.createElement('div');
                dropdown.classList.add('custom-select-dropdown');
                dropdown.style.maxHeight = '100px';
                dropdown.style.overflowY = 'auto';
                dropdown.style.border = '1px solid #ccc';
                dropdown.style.borderRadius = '4px';
                dropdown.style.backgroundColor = '#fff';
                dropdown.style.position = 'absolute';
                dropdown.style.width = '100%';
                dropdown.style.zIndex = '1000';
                dropdown.style.display = 'none'; // Ocultar inicialmente

                // Clonar las opciones del select original
                const options = Array.from(select.options);
                const nonPlaceholderOptions = options.filter(option => option.value.trim() !== '');
                nonPlaceholderOptions.sort((a, b) => a.text.localeCompare(b.text));

                // Vaciar el select y agregar las opciones ordenadas
                jQuery(select).empty();
                jQuery(select).append(new Option('Seleccionar...', '', false, false)); // Placeholder

                nonPlaceholderOptions.forEach(option => {
                    jQuery(select).append(new Option(option.text, option.value));
                });

                // Agregar las opciones al dropdown
                const renderDropdownOptions = (filter = '') => {
                    dropdown.innerHTML = ''; // Limpiar el dropdown
                    const filteredOptions = nonPlaceholderOptions.filter(option =>
                        option.text.toLowerCase().includes(filter.toLowerCase())
                    );

                    filteredOptions.forEach(option => {
                        const optionElement = document.createElement('div');
                        optionElement.textContent = option.text;
                        optionElement.style.padding = '8px';
                        optionElement.style.cursor = 'pointer';
                        optionElement.style.backgroundColor = '#fff';
                        optionElement.dataset.value = option.value;

                        // Resaltar la opción si coincide con el valor seleccionado
                        if (option.value === select.value) {
                            optionElement.style.backgroundColor = '#f0f0f0';
                        }

                        optionElement.addEventListener('click', () => {
                            select.value = option.value;
                            searchInput.value = option.text; // Actualizar el campo de búsqueda
                            select.dispatchEvent(new Event('change')); // Disparar el evento change
                            dropdown.style.display = 'none'; // Ocultar el dropdown
                        });

                        dropdown.appendChild(optionElement);
                    });
                };

                // Renderizar las opciones iniciales
                renderDropdownOptions();

                // Mostrar/ocultar el dropdown al hacer clic en el campo de búsqueda
                searchInput.addEventListener('click', (e) => {
                    e.stopPropagation();
                    dropdown.style.display = dropdown.style.display === 'none' ? 'block' : 'none';
                });

                // Filtrar las opciones según el texto ingresado en el buscador
                searchInput.addEventListener('input', () => {
                    renderDropdownOptions(searchInput.value);
                    dropdown.style.display = 'block'; // Asegurarse de que el dropdown esté visible
                });

                // Ocultar el dropdown al hacer clic fuera
                document.addEventListener('click', () => {
                    dropdown.style.display = 'none';
                });

                // Envolver el select original
                select.parentNode.insertBefore(wrapper, select);
                wrapper.appendChild(searchInput);
                wrapper.appendChild(dropdown);

                // Marcar como inicializado
                select.setAttribute('data-custom-select-initialized', 'true');
            }
        };

        // Función para manejar el evento de selección en el select
        const handleSelectChange = () => {
            const select = document.querySelector('.jet-abaf-details__field-apartment_id select');
            if (select) {
                const selectedValue = select.value;
                if (selectedValue) {
                    console.log('Booking item selected:', selectedValue);

                    // Activar el campo de fecha solo si el select tiene valor
                    const dateFields = document.querySelector('.jet-abaf-details__booking-dates');
                    if (dateFields && dateFields.classList.contains('jet-abaf-disabled')) {
                        dateFields.classList.remove('jet-abaf-disabled');
                        console.log('Re-enabled the date picker fields.');
                    }
                }
            }
        };

        // Configuración del observador para detectar cambios dinámicos en el DOM
        const observer = new MutationObserver(mutations => {
            mutations.forEach(mutation => {
                if (mutation.addedNodes.length) {
                    const select = document.querySelector('.jet-abaf-details__field-apartment_id select');
                    if (select && !select.hasAttribute('data-custom-select-initialized')) {
                        enhanceSelectWithScrollAndSearch(); // Aplicar la mejora al select
                    }

                    const dateFields = document.querySelector('.jet-abaf-details__booking-dates');
                    if (dateFields && dateFields.classList.contains('jet-abaf-disabled')) {
                        // Reactivar las fechas si es necesario
                        const bookingItemSelect = document.querySelector('.jet-abaf-details__field-apartment_id select');
                        if (bookingItemSelect && bookingItemSelect.value) {
                            handleSelectChange(); // Activar las fechas si es necesario
                        }
                    }
                }
            });
        });

        // Función para detectar el cambio en el select
        const setupSelectChangeListener = () => {
            const select = document.querySelector('.jet-abaf-details__field-apartment_id select');
            if (select) {
                select.addEventListener('change', handleSelectChange);
            }
        };

        // Al cargar el DOM, inicializar todo el proceso
        document.addEventListener('DOMContentLoaded', function () {
            try {
                // Inicializar el select con scroll horizontal, alto fijo y buscador
                enhanceSelectWithScrollAndSearch();

                // Establecer el observador para detectar cambios dinámicos
                observer.observe(document.body, { childList: true, subtree: true });

                // Configurar el listener para detectar cambios en el select
                setupSelectChangeListener();
            } catch (error) {
                console.error('Error processing the script:', error);
            }
        });
    </script>

    <style>
        /* Estilo para el select */
        .jet-abaf-details__field-apartment_id select {
            display: none; /* Ocultar el select original */
        }

        /* Estilo para el contenedor personalizado */
        .custom-select-wrapper {
            position: relative;
            width: 100%;
        }

        /* Estilo para el campo de búsqueda */
        .custom-select-wrapper input[type="text"] {
            padding: 8px;
            margin-bottom: 5px;
            width: 100%;
            box-sizing: border-box;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        /* Estilo para el dropdown */
        .custom-select-dropdown {
            max-height: 100px;
            overflow-y: auto;
            border: 1px solid #ccc;
            border-radius: 4px;
            background-color: #fff;
            position: absolute;
            width: 100%;
            z-index: 1000;
        }

        /* Estilo para las opciones del dropdown */
        .custom-select-dropdown div {
            padding: 8px;
            cursor: pointer;
        }

        .custom-select-dropdown div:hover {
            background-color: #f0f0f0;
        }

        /* Estilo para la opción seleccionada */
        .custom-select-dropdown div.selected {
            background-color: #d3d3d3;
            font-weight: bold;
        }
    </style>
    <?php
});

