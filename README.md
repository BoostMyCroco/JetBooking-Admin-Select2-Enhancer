# JetBooking-Admin-Select2-Enhancer

## Description

This script improves the user experience within the JetBooking admin popup by:
- Sorting `<select>` options alphabetically.
- Applying the Select2 library for a modern, searchable dropdown.
- Ensuring the dropdown works seamlessly inside modals or popups.

## Features

- **Alphabetical Sorting**: Automatically sorts dropdown options in ascending order.
- **Select2 Integration**: Provides a clean and searchable interface for dropdowns.
- **Popup Compatibility**: Ensures proper rendering within JetBooking modals.

## Installation

1. Copy the script into your WordPress installation.
2. Include the script in your theme's `functions.php` file or in a custom plugin.
3. Make sure to have the Select2 library included via the provided enqueue functions.

## Usage

1. This script targets dropdowns within the JetBooking admin interface, specifically those with the class `.jet-abaf-details__field-apartment_id select`.
2. The dropdown will automatically enhance itself with alphabetical sorting and Select2 functionality whenever the modal or popup loads.

## Dependencies

jQuery: Ensure jQuery is loaded in your WordPress installation.
Select2: The script enqueues Select2 (v4.0.13) automatically.

## Customization

If you need to modify the dropdown parent or other Select2 options:

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
add_action('admin_enqueue_scripts', function () {
    wp_enqueue_style('select2-css', 'https://cdnjs.cloudflare.com/ajax/libs/select2/4.0.13/css/select2.min.css');
    wp_enqueue_script('select2-js', 'https://cdnjs.cloudflare.com/ajax/libs/select2/4.0.13/js/select2.min.js', ['jquery'], null, true);
});

add_action('admin_footer', function () {
    ?>
    <script>
        const fixDisabledDates = () => {
            const dateFields = document.querySelector('.jet-abaf-details__booking-dates');
            if (dateFields && dateFields.classList.contains('jet-abaf-disabled')) {
                dateFields.classList.remove('jet-abaf-disabled');
                console.log('Re-enabled the date picker fields.');

                // Simulate a change event on the Booking Item select to trigger JetBooking's internal logic
                const bookingItemSelect = document.querySelector('.jet-abaf-details__field-apartment_id select');
                if (bookingItemSelect) {
                    const event = new Event('change', { bubbles: true });
                    bookingItemSelect.dispatchEvent(event);
                    console.log('Triggered change event on Booking Item select.');
                }
            }
        };

        document.addEventListener('DOMContentLoaded', function () {
            try {
                const enhanceSelectWithSelect2 = () => {
                    const select = document.querySelector('.jet-abaf-details__field-apartment_id select');

                    if (select && !select.hasAttribute('data-select2-initialized')) {
                        const options = Array.from(select.options);
                        const placeholder = options.find(option => !option.value || option.value.trim() === '');
                        const nonPlaceholderOptions = options.filter(option => option.value && option.value.trim() !== '');

                        nonPlaceholderOptions.sort((a, b) => a.text.localeCompare(b.text));

                        jQuery(select).empty();
                        if (placeholder) {
                            jQuery(select).append(new Option(placeholder.text || 'Select an option', '', true, true));
                        }
                        nonPlaceholderOptions.forEach(option => {
                            jQuery(select).append(new Option(option.text, option.value));
                        });

                        if (jQuery(select).data('select2')) {
                            jQuery(select).select2('destroy');
                        }
                        jQuery(select).select2({
                            placeholder: placeholder ? placeholder.text : 'Select an option',
                            allowClear: true,
                            width: '100%',
                            dropdownParent: jQuery('.jet-abaf-popup'),
                        });

                        select.setAttribute('data-select2-initialized', 'true');
                        fixDisabledDates(); // Ensure date pickers are re-enabled
                    }
                };

                const observer = new MutationObserver(mutations => {
                    mutations.forEach(mutation => {
                        if (mutation.addedNodes.length) {
                            const select = document.querySelector('.jet-abaf-details__field-apartment_id select');
                            if (select) {
                                enhanceSelectWithSelect2();
                                fixDisabledDates(); // Ensure dates are re-enabled on DOM updates
                            }
                        }
                    });
                });

                observer.observe(document.body, { childList: true, subtree: true });

                enhanceSelectWithSelect2();
            } catch (error) {
                console.error('Error processing the script:', error);
            }
        });
    </script>
    <?php
});

