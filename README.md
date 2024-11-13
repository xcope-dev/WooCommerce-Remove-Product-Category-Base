# WooCommerce-Remove-Product-Category-Base
Usuwa kategorię produktów z URL w WooCommerce, upraszczając strukturę linków dla lepszego SEO.


## Opis
Ten fragment kodu PHP umożliwia usunięcie domyślnego elementu „kategoria-produktu” z URL kategorii WooCommerce. Adresy URL stają się krótsze i bardziej przyjazne dla użytkowników oraz wyszukiwarek, co może wpłynąć korzystnie na SEO.

Kod najlepiej umieścić w pliku `functions.php` aktywnego motywu WordPress, z którego korzysta Twój sklep WooCommerce.

## Funkcje kodu
- Usuwa z URL kategorii frazę "kategoria-produktu".
- Automatycznie aktualizuje reguły przekierowań, tworząc przyjazne dla użytkownika URL dla każdej kategorii i jej podkategorii.

## Instalacja
1. Skopiuj kod do pliku `functions.php` aktywnego motywu WordPress w Twojej witrynie.
2. Zapisz plik.
3. Po wprowadzeniu zmian warto przejść do „Ustawienia > Bezpośrednie odnośniki” w panelu administracyjnym WordPress i zapisać ustawienia (bez wprowadzania zmian), aby zaktualizować reguły przekierowań.

## Kod
```php

/**
 * Ustawienie odnośników
 */

// Usuwa 'kategoria-produktu' z URL kategorii WooCommerce
add_filter('woocommerce_product_category_permalink', 'remove_product_category_base');
function remove_product_category_base($category_permalink) {
    $category_permalink = str_replace('/kategoria-produktu/', '/', $category_permalink);
    return $category_permalink;
}

// Aktualizuje rewrite rules
add_action('init', 'custom_remove_category_base');
function custom_remove_category_base() {
    // Pobiera wszystkie kategorie produktów
    $terms = get_terms(array(
        'taxonomy' => 'product_cat',
        'hide_empty' => false,
    ));

    if (!is_wp_error($terms) && !empty($terms)) {
        foreach ($terms as $term) {
            // Dodaje regułę dla każdej kategorii i jej podkategorii
            add_rewrite_rule(
                '^' . $term->slug . '/?$',
                'index.php?product_cat=' . $term->slug,
                'top'
            );

            // Pobiera podkategorie dla bieżącej kategorii
            $children = get_terms(array(
                'taxonomy' => 'product_cat',
                'hide_empty' => false,
                'parent' => $term->term_id,
            ));

            if (!is_wp_error($children) && !empty($children)) {
                foreach ($children as $child) {
                    add_rewrite_rule(
                        '^' . $term->slug . '/' . $child->slug . '/?$',
                        'index.php?product_cat=' . $child->slug,
                        'top'
                    );
                }
            }
        }
    }
