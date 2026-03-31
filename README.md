Fix: Wazuh Dashboard illegal_argument_exception for manager.name
This guide provides a permanent solution for the common Wazuh error where text fields (like manager.name) are not optimized for aggregations and sorting.

The Problem
After updating Wazuh or modifying index templates, fields that should be keyword might be dynamically mapped as text. This causes the following error in the Wazuh Dashboard:

[WazuhError]: search_phase_execution_exception: [illegal_argument_exception] Reason: Text fields are not optimised for operations that require per-document field data like aggregations and sorting...

The Solution

Phase 1: Fixing Future Indices (Permanent Fix)
We need to create a high-priority index template to force manager.name back to the keyword type. Run this in the Wazuh Dashboard > Dev Tools:

JSON
PUT /_index_template/wazuh_fix_manager
{
  "index_patterns": ["wazuh-alerts-4.x-*"],
  "priority": 500,
  "template": {
    "mappings": {
      "properties": {
        "manager": {
          "properties": {
            "name": {
              "type": "keyword",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        }
      }
    }
  }
}

Phase 2: Fixing Current/Existing Indices

The template above only affects new indices. For indices that already exist and show the error (e.g., wazuh-alerts-4.x-2026.01.05), you must enable fielddata as a temporary bridge:

JSON
PUT /wazuh-alerts-*/_mapping
{
  "properties": {
    "manager.name": {
      "type": "text",
      "fielddata": true
    }
  }
}

Note: If you get a "cannot change from keyword to text" error, it means that specific index is already fixed. You can run this command for specific dates if the wildcard * fails.

Phase 3: Refresh Index Pattern

Go to Stack Management > Index Patterns.

Select wazuh-alerts-*.

Click the Refresh field list (circular arrow) button in the top right corner.

Why this happens?
Wazuh expects certain fields to be keyword for dashboard visualizations. If a template conflict occurs, OpenSearch/Elasticsearch defaults to text mapping, which prevents sorting and grouping to protect memory performance. By setting a priority: 500 template, we ensure our correct mapping overrides any faulty defaults.
