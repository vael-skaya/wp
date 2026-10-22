# FASE — PERBANDINGAN DATABASE PRODUKSI vs PEMULIHAN LOKAL

**22 Oktober 2026** · **READ-ONLY murni** — tanpa import/DROP/UPDATE/ALTER/migrasi · tanpa akses hosting

**Sumber produksi:** dump SQL cPanel di `D:\backup-database-smk10november\` (bukan ekspor baru) ✓

> ⚠️ Dump berasal dari lingkungan yang **pernah dikompromikan** — tabel di produksi bukan otomatis layak dipertahankan.

## Ringkasan
| Sekolah | Dump | Ukuran | Tabel produksi | Tabel staging | Hanya di produksi | Hanya di staging |
|---|---|---:|---:|---:|---:|---:|
| Tambun | `smkq8953_utama1.sql` | 76.8 MB | **132** | **24** | **111** | 3 |
| Cibitung | `smkq8953_cbitung.sql` | 81.4 MB | **131** | **24** | **110** | 3 |
| Kebumen | `smkq8953_kebumen.sql` | 91.4 MB | **138** | **13** | **125** | 0 |
| Sragen | `smkq8953_sragen10.sql` | 77.1 MB | **132** | **13** | **119** | 0 |

## Tambun

**Hanya di produksi (111):** `cartflows_ca_cart_abandonment`, `cartflows_ca_email_history`, `cartflows_ca_email_templates`, `cartflows_ca_email_templates_meta`, `e_notes`, `e_notes_users_relations`, `e_submissions`, `e_submissions_actions_log`, `e_submissions_values`, `frmt_form_entry`, `frmt_form_entry_meta`, `frmt_form_reports`, `frmt_form_views`, `gla_attribute_mapping_rules`, `gla_budget_recommendations`, `gla_merchant_issues`, `gla_merchant_price_benchmarks`, `gla_shipping_rates`, `gla_shipping_times`, `hostinger_reach_carts`, `hostinger_reach_contact_lists`, `hostinger_reach_forms`, `jet_apartment_bookings`, `jet_apartment_units`, `jet_bookings_meta`, `jet_post_types`, `jet_smart_filters_indexer`, `jet_taxonomies`, `jetpack_sync_queue`, `mailpoet_automation_run_logs`, `mailpoet_automation_run_subjects`, `mailpoet_automation_runs`, `mailpoet_automation_triggers`, `mailpoet_automation_versions`, `mailpoet_automations`, `mailpoet_custom_fields`, `mailpoet_dynamic_segment_filters`, `mailpoet_feature_flags`, `mailpoet_forms`, `mailpoet_log`, `mailpoet_migrations`, `mailpoet_newsletter_links`, `mailpoet_newsletter_option`, `mailpoet_newsletter_option_fields`, `mailpoet_newsletter_posts`, `mailpoet_newsletter_segment`, `mailpoet_newsletter_templates`, `mailpoet_newsletters`, `mailpoet_scheduled_task_subscribers`, `mailpoet_scheduled_tasks`, `mailpoet_segments`, `mailpoet_sending_queues`, `mailpoet_settings`, `mailpoet_statistics_bounces`, `mailpoet_statistics_clicks`, `mailpoet_statistics_forms`, `mailpoet_statistics_newsletters`, `mailpoet_statistics_opens`, `mailpoet_statistics_unsubscribes`, `mailpoet_statistics_woocommerce_purchases`, `mailpoet_stats_notifications`, `mailpoet_subscriber_custom_field`, `mailpoet_subscriber_ips`, `mailpoet_subscriber_segment`, `mailpoet_subscriber_tag`, `mailpoet_subscribers`, `mailpoet_tags`, `mailpoet_user_agents`, `mailpoet_user_flags`, `snippets`, `wc_admin_note_actions`, `wc_admin_notes`, `wc_category_lookup`, `wc_customer_lookup`, `wc_download_log`, `wc_order_addresses`, `wc_order_coupon_lookup`, `wc_order_operational_data`, `wc_order_product_lookup`, `wc_order_stats`, `wc_order_tax_lookup`, `wc_orders`, `wc_orders_meta`, `wc_product_attributes_lookup`, `wc_product_download_directories`, `wc_product_meta_lookup`, `wc_rate_limits`, `wc_reserved_stock`, `wc_tax_rate_classes`, `wc_webhooks`, `woocommerce_api_keys`, `woocommerce_attribute_taxonomies`, `woocommerce_downloadable_product_permissions`, `woocommerce_log`, `woocommerce_order_itemmeta`, `woocommerce_order_items`, `woocommerce_payment_tokenmeta`, `woocommerce_payment_tokens`, `woocommerce_sessions`, `woocommerce_shipping_zone_locations`, `woocommerce_shipping_zone_methods`, `woocommerce_shipping_zones`, `woocommerce_tax_rate_locations`, `woocommerce_tax_rates`, `woof_sd`, `woof_sd_presets`, `wpf_filters`, `wpf_meta_keys`, `wpf_meta_values`, `wpf_meta_values_bk`, `wpfm_backup`


**Hanya di staging (3):** `wpforms_analytics_forms`, `wpforms_analytics_snapshots`, `wpforms_product_events_queue`


**Baris di staging:** `wp_actionscheduler_actions`=53, `wp_actionscheduler_groups`=3, `wp_actionscheduler_logs`=145, `wp_options`=220, `wp_postmeta`=547, `wp_posts`=166, `wp_term_relationships`=26, `wp_term_taxonomy`=9, `wp_terms`=9, `wp_usermeta`=15, `wp_users`=1, `wp_wpforms_tasks_meta`=4

## Cibitung

**Hanya di produksi (110):** `cartflows_ca_cart_abandonment`, `cartflows_ca_email_history`, `cartflows_ca_email_templates`, `cartflows_ca_email_templates_meta`, `e_notes`, `e_notes_users_relations`, `e_submissions`, `e_submissions_actions_log`, `e_submissions_values`, `frmt_form_entry`, `frmt_form_entry_meta`, `frmt_form_reports`, `frmt_form_views`, `gla_attribute_mapping_rules`, `gla_budget_recommendations`, `gla_merchant_issues`, `gla_merchant_price_benchmarks`, `gla_shipping_rates`, `gla_shipping_times`, `hostinger_reach_carts`, `hostinger_reach_contact_lists`, `hostinger_reach_forms`, `jet_apartment_bookings`, `jet_apartment_units`, `jet_bookings_meta`, `jet_post_types`, `jet_smart_filters_indexer`, `jet_taxonomies`, `jetpack_sync_queue`, `mailpoet_automation_run_logs`, `mailpoet_automation_run_subjects`, `mailpoet_automation_runs`, `mailpoet_automation_triggers`, `mailpoet_automation_versions`, `mailpoet_automations`, `mailpoet_custom_fields`, `mailpoet_dynamic_segment_filters`, `mailpoet_feature_flags`, `mailpoet_forms`, `mailpoet_log`, `mailpoet_migrations`, `mailpoet_newsletter_links`, `mailpoet_newsletter_option`, `mailpoet_newsletter_option_fields`, `mailpoet_newsletter_posts`, `mailpoet_newsletter_segment`, `mailpoet_newsletter_templates`, `mailpoet_newsletters`, `mailpoet_scheduled_task_subscribers`, `mailpoet_scheduled_tasks`, `mailpoet_segments`, `mailpoet_sending_queues`, `mailpoet_settings`, `mailpoet_statistics_bounces`, `mailpoet_statistics_clicks`, `mailpoet_statistics_forms`, `mailpoet_statistics_newsletters`, `mailpoet_statistics_opens`, `mailpoet_statistics_unsubscribes`, `mailpoet_statistics_woocommerce_purchases`, `mailpoet_stats_notifications`, `mailpoet_subscriber_custom_field`, `mailpoet_subscriber_ips`, `mailpoet_subscriber_segment`, `mailpoet_subscriber_tag`, `mailpoet_subscribers`, `mailpoet_tags`, `mailpoet_user_agents`, `mailpoet_user_flags`, `wc_admin_note_actions`, `wc_admin_notes`, `wc_category_lookup`, `wc_customer_lookup`, `wc_download_log`, `wc_order_addresses`, `wc_order_coupon_lookup`, `wc_order_operational_data`, `wc_order_product_lookup`, `wc_order_stats`, `wc_order_tax_lookup`, `wc_orders`, `wc_orders_meta`, `wc_product_attributes_lookup`, `wc_product_download_directories`, `wc_product_meta_lookup`, `wc_rate_limits`, `wc_reserved_stock`, `wc_tax_rate_classes`, `wc_webhooks`, `woocommerce_api_keys`, `woocommerce_attribute_taxonomies`, `woocommerce_downloadable_product_permissions`, `woocommerce_log`, `woocommerce_order_itemmeta`, `woocommerce_order_items`, `woocommerce_payment_tokenmeta`, `woocommerce_payment_tokens`, `woocommerce_sessions`, `woocommerce_shipping_zone_locations`, `woocommerce_shipping_zone_methods`, `woocommerce_shipping_zones`, `woocommerce_tax_rate_locations`, `woocommerce_tax_rates`, `woof_sd`, `woof_sd_presets`, `wpf_filters`, `wpf_meta_keys`, `wpf_meta_values`, `wpf_meta_values_bk`, `wpfm_backup`


**Hanya di staging (3):** `wpforms_analytics_forms`, `wpforms_analytics_snapshots`, `wpforms_product_events_queue`


**Baris di staging:** `wp_actionscheduler_actions`=58, `wp_actionscheduler_groups`=3, `wp_actionscheduler_logs`=160, `wp_options`=226, `wp_postmeta`=640, `wp_posts`=205, `wp_term_relationships`=25, `wp_term_taxonomy`=9, `wp_terms`=9, `wp_usermeta`=15, `wp_users`=1, `wp_wpforms_tasks_meta`=4

## Kebumen

**Hanya di produksi (125):** `actionscheduler_actions`, `actionscheduler_claims`, `actionscheduler_groups`, `actionscheduler_logs`, `ahm_asset_links`, `ahm_assets`, `ahm_cron_jobs`, `ahm_download_stats`, `ahm_emails`, `ahm_sessions`, `ahm_social_conns`, `ahm_user_download_counts`, `cartflows_ca_cart_abandonment`, `cartflows_ca_email_history`, `cartflows_ca_email_templates`, `cartflows_ca_email_templates_meta`, `e_notes`, `e_notes_users_relations`, `e_submissions`, `e_submissions_actions_log`, `e_submissions_values`, `frmt_form_entry`, `frmt_form_entry_meta`, `frmt_form_reports`, `frmt_form_views`, `gla_attribute_mapping_rules`, `gla_budget_recommendations`, `gla_merchant_issues`, `gla_merchant_price_benchmarks`, `gla_shipping_rates`, `gla_shipping_times`, `hostinger_reach_carts`, `hostinger_reach_contact_lists`, `hostinger_reach_forms`, `jet_apartment_bookings`, `jet_apartment_units`, `jet_bookings_meta`, `jet_post_types`, `jet_smart_filters_indexer`, `jet_taxonomies`, `jetpack_sync_queue`, `mailpoet_automation_run_logs`, `mailpoet_automation_run_subjects`, `mailpoet_automation_runs`, `mailpoet_automation_triggers`, `mailpoet_automation_versions`, `mailpoet_automations`, `mailpoet_custom_fields`, `mailpoet_dynamic_segment_filters`, `mailpoet_feature_flags`, `mailpoet_forms`, `mailpoet_log`, `mailpoet_migrations`, `mailpoet_newsletter_links`, `mailpoet_newsletter_option`, `mailpoet_newsletter_option_fields`, `mailpoet_newsletter_posts`, `mailpoet_newsletter_segment`, `mailpoet_newsletter_templates`, `mailpoet_newsletters`, `mailpoet_scheduled_task_subscribers`, `mailpoet_scheduled_tasks`, `mailpoet_segments`, `mailpoet_sending_queues`, `mailpoet_settings`, `mailpoet_statistics_bounces`, `mailpoet_statistics_clicks`, `mailpoet_statistics_forms`, `mailpoet_statistics_newsletters`, `mailpoet_statistics_opens`, `mailpoet_statistics_unsubscribes`, `mailpoet_statistics_woocommerce_purchases`, `mailpoet_stats_notifications`, `mailpoet_subscriber_custom_field`, `mailpoet_subscriber_ips`, `mailpoet_subscriber_segment`, `mailpoet_subscriber_tag`, `mailpoet_subscribers`, `mailpoet_tags`, `mailpoet_user_agents`, `mailpoet_user_flags`, `wc_admin_note_actions`, `wc_admin_notes`, `wc_category_lookup`, `wc_customer_lookup`, `wc_download_log`, `wc_order_addresses`, `wc_order_coupon_lookup`, `wc_order_operational_data`, `wc_order_product_lookup`, `wc_order_stats`, `wc_order_tax_lookup`, `wc_orders`, `wc_orders_meta`, `wc_product_attributes_lookup`, `wc_product_download_directories`, `wc_product_meta_lookup`, `wc_rate_limits`, `wc_reserved_stock`, `wc_tax_rate_classes`, `wc_webhooks`, `woocommerce_api_keys`, `woocommerce_attribute_taxonomies`, `woocommerce_downloadable_product_permissions`, `woocommerce_log`, `woocommerce_order_itemmeta`, `woocommerce_order_items`, `woocommerce_payment_tokenmeta`, `woocommerce_payment_tokens`, `woocommerce_sessions`, `woocommerce_shipping_zone_locations`, `woocommerce_shipping_zone_methods`, `woocommerce_shipping_zones`, `woocommerce_tax_rate_locations`, `woocommerce_tax_rates`, `woof_sd`, `woof_sd_presets`, `wpf_filters`, `wpf_meta_keys`, `wpf_meta_values`, `wpf_meta_values_bk`, `wpforms_logs`, `wpforms_payment_meta`, `wpforms_payments`, `wpforms_tasks_meta`


**Hanya di staging (0):** —


**Baris di staging:** `wp_comments`=1, `wp_options`=183, `wp_postmeta`=590, `wp_posts`=187, `wp_term_relationships`=39, `wp_term_taxonomy`=12, `wp_terms`=12, `wp_usermeta`=16, `wp_users`=1

## Sragen

**Hanya di produksi (119):** `actionscheduler_actions`, `actionscheduler_claims`, `actionscheduler_groups`, `actionscheduler_logs`, `cartflows_ca_cart_abandonment`, `cartflows_ca_email_history`, `cartflows_ca_email_templates`, `cartflows_ca_email_templates_meta`, `e_notes`, `e_notes_users_relations`, `e_submissions`, `e_submissions_actions_log`, `e_submissions_values`, `frmt_form_entry`, `frmt_form_entry_meta`, `frmt_form_reports`, `frmt_form_views`, `gla_attribute_mapping_rules`, `gla_budget_recommendations`, `gla_merchant_issues`, `gla_merchant_price_benchmarks`, `gla_shipping_rates`, `gla_shipping_times`, `hostinger_reach_carts`, `hostinger_reach_contact_lists`, `hostinger_reach_forms`, `jet_apartment_bookings`, `jet_apartment_units`, `jet_bookings_meta`, `jet_post_types`, `jet_smart_filters_indexer`, `jet_taxonomies`, `jetpack_sync_queue`, `mailpoet_automation_run_logs`, `mailpoet_automation_run_subjects`, `mailpoet_automation_runs`, `mailpoet_automation_triggers`, `mailpoet_automation_versions`, `mailpoet_automations`, `mailpoet_custom_fields`, `mailpoet_dynamic_segment_filters`, `mailpoet_feature_flags`, `mailpoet_forms`, `mailpoet_log`, `mailpoet_migrations`, `mailpoet_newsletter_links`, `mailpoet_newsletter_option`, `mailpoet_newsletter_option_fields`, `mailpoet_newsletter_posts`, `mailpoet_newsletter_segment`, `mailpoet_newsletter_templates`, `mailpoet_newsletters`, `mailpoet_scheduled_task_subscribers`, `mailpoet_scheduled_tasks`, `mailpoet_segments`, `mailpoet_sending_queues`, `mailpoet_settings`, `mailpoet_statistics_bounces`, `mailpoet_statistics_clicks`, `mailpoet_statistics_forms`, `mailpoet_statistics_newsletters`, `mailpoet_statistics_opens`, `mailpoet_statistics_unsubscribes`, `mailpoet_statistics_woocommerce_purchases`, `mailpoet_stats_notifications`, `mailpoet_subscriber_custom_field`, `mailpoet_subscriber_ips`, `mailpoet_subscriber_segment`, `mailpoet_subscriber_tag`, `mailpoet_subscribers`, `mailpoet_tags`, `mailpoet_user_agents`, `mailpoet_user_flags`, `snippets`, `wc_admin_note_actions`, `wc_admin_notes`, `wc_category_lookup`, `wc_customer_lookup`, `wc_download_log`, `wc_order_addresses`, `wc_order_coupon_lookup`, `wc_order_operational_data`, `wc_order_product_lookup`, `wc_order_stats`, `wc_order_tax_lookup`, `wc_orders`, `wc_orders_meta`, `wc_product_attributes_lookup`, `wc_product_download_directories`, `wc_product_meta_lookup`, `wc_rate_limits`, `wc_reserved_stock`, `wc_tax_rate_classes`, `wc_webhooks`, `woocommerce_api_keys`, `woocommerce_attribute_taxonomies`, `woocommerce_downloadable_product_permissions`, `woocommerce_log`, `woocommerce_order_itemmeta`, `woocommerce_order_items`, `woocommerce_payment_tokenmeta`, `woocommerce_payment_tokens`, `woocommerce_sessions`, `woocommerce_shipping_zone_locations`, `woocommerce_shipping_zone_methods`, `woocommerce_shipping_zones`, `woocommerce_tax_rate_locations`, `woocommerce_tax_rates`, `woof_sd`, `woof_sd_presets`, `wpf_filters`, `wpf_meta_keys`, `wpf_meta_values`, `wpf_meta_values_bk`, `wpfm_backup`, `wpforms_logs`, `wpforms_payment_meta`, `wpforms_payments`, `wpforms_tasks_meta`


**Hanya di staging (0):** —


**Baris di staging:** `wp_options`=177, `wp_postmeta`=506, `wp_posts`=152, `wp_term_relationships`=26, `wp_term_taxonomy`=9, `wp_terms`=9, `wp_usermeta`=15, `wp_users`=1

## Penilaian
- **Data inti WordPress** (posts, postmeta, options, users, umeta, terms, taxonomy, relationships, comments) ada di **kedua sisi** ✓
- Produksi memuat **111–130 tabel plugin** (Forminator `e_submissions*`, Fluent Forms `frmt_*`, CartFlows, Action Scheduler, GLA, dll.) yang **tidak dibawa** ke staging
- Staging memuat beberapa tabel plugin **baru** (`actionscheduler_*`, `wpforms_*`, `e_events`) — bukan kehilangan ✓
- **Jumlah baris produksi tidak dihitung** (butuh import ✗ dilarang); hanya jumlah statement `INSERT` sebagai indikator kasar ✓

## Dampak yang mungkin
| Kategori | Risiko | Catatan |
|---|---|---|
| **Entri formulir / pendaftaran** (`e_submissions`, `frmt_form_entry`) | **TINGGI** | data pendaftaran siswa/pertanyaan warga bisa hilang |
| Cart abandonment / e-commerce | Rendah | kemungkinan tidak dipakai situs sekolah |
| Action Scheduler / log | Rendah | bersifat sementara |
| Newsletter / email | Rendah-Sedang | daftar bisa hilang |
| Akun pengguna lama | **JANGAN dimigrasikan** | 163 akun palsu dari serangan |

## Tindakan minimum sebelum go-live
1. Tentukan per tabel "hanya produksi": **migrasi / abaikan / karantina** — berdasarkan isi, bukan nama.
2. Untuk tabel formulir: ekspor **hanya tabel itu** dari produksi, kirim ke saya untuk dianalisis **tanpa diimpor**.
3. **Jangan** migrasikan `users`/`usermeta`/`options` dari produksi.
4. Tinjau keamanan isi setiap tabel plugin (bisa memuat injeksi penyerang) sebelum dipakai.

## Keterbatasan jujur
- Perbandingan **struktur tabel**, bukan isi/kolom/index secara rinci ✗
- Jumlah baris produksi **tidak diketahui** ✗
- **Tidak** menyatakan situs siap live ✓
- Catatan proses: percobaan awal gagal karena bash tidak bisa membaca drive `D:`; parsing akhir via Python ✓
