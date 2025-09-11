---
Vulnerability: XSS
Category: Payload
---
# XSS Payloads Index

This index references context-specific payload files.

- [# XSS Payloads by Rendering Context (Responsible, PoC-focused)](split/xss_payloads_by_rendering_context_responsible_poc-focused.md)
- [Legend](split/legend.md)
- [1) HTML **Text** context (between tags)](split/1_html_text_context_between_tags.md)
- [2) HTML **Attribute** context (quoted)](split/2_html_attribute_context_quoted.md)
- [3) HTML **Attribute** context (unquoted)](split/3_html_attribute_context_unquoted.md)
- [4) **Event handler attribute** (on\*)](split/4_event_handler_attribute_on.md)
- [5) **JavaScript string** context (inside `<script>` as a string)](split/5_javascript_string_context_inside_script_as_a_string.md)
- [6) **Raw JavaScript** context (not in a string)](split/6_raw_javascript_context_not_in_a_string.md)
- [7) **URL attribute** context (`href`, `src`, `action`, `formaction`)](split/7_url_attribute_context_href_src_action_formaction.md)
- [8) **Style** / CSS context](split/8_style_css_context.md)
- [9) **SVG / MathML** context](split/9_svg_mathml_context.md)
- [10) **HTML comment** context](split/10_html_comment_context.md)
- [11) **JSON in `<script type="application/json">`**](split/11_json_in_script_type_application_json.md)
- [12) **DOM insertion sinks** (client-side rendering)](split/12_dom_insertion_sinks_client-side_rendering.md)
- [13) **URL fragment / hash** (often used in SPAs)](split/13_url_fragment_hash_often_used_in_spas.md)
- [14) **postMessage** / cross-document messaging](split/14_postmessage_cross-document_messaging.md)
- [15) **Email/Report renderers** (HTML emails, PDF/HTML exports)](split/15_email_report_renderers_html_emails_pdf_html_exports.md)
- [Non-executable markers (good for diffing safe vs unsafe)](split/non-executable_markers_good_for_diffing_safe_vs_unsafe.md)
- [What **not** to do in testing](split/what_not_to_do_in_testing.md)
- [Quick cross-check when a payload "doesn't fire"](split/quick_cross-check_when_a_payload_doesn_t_fire.md)
