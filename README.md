# River Flood Early Warning

This n8n template checks seven-day river-discharge forecasts from the Open-Meteo Flood API every six hours, evaluates configured river points, and asks a local Ollama model for a short plain-language bulletin. It can post the bulletin to Slack only when a point is at risk and a webhook URL has been configured.

## Import

1. In n8n, open **Workflows → Import from file** and select `river-flood-early-warning.json`.
2. Open **Ollama Chat Model** and select or create an `ollamaApi` credential. The exported workflow contains only the placeholder credential reference `Ollama account`; it contains no credential values.
3. Save the workflow. The Schedule Trigger is set to every six hours, and the Manual Trigger is available for testing.

If n8n runs in Docker and Ollama runs on the host, set the credential's base URL to `http://host.docker.internal:11434`.

Tested with n8n 2.42.7 and Ollama 0.40 running `gemma4:e4b-it-qat`.

## Configuration

Edit **Configuration**, the single editable configuration source. It contains:

- `stations`: station `name`, `latitude`, `longitude`, and example `alert_m3s` values for Usumacinta at Boca del Cerro and Grijalva near Villahermosa in Tabasco, Mexico.
- `rise_alert_pct`: the percentage rise over today's modeled discharge that also marks a station at risk.
- `bulletin_language`: the bulletin language, English by default.
- `ollama_model`: the local model name, `gemma3:4b` by default. Pull it with `ollama pull gemma3:4b` before running.
- `slack_webhook_url`: empty by default. Leave it empty to skip Slack, or provide a Slack Incoming Webhook URL.

The threshold values are examples, not official limits. Replace them with values appropriate for the local water authority and station.

## How it works

The workflow expands the station list, makes one Open-Meteo Flood API request per station, and evaluates today's discharge, the maximum seven-day discharge, its date, and `at_risk`. Missing or malformed station data becomes `data_missing: true` and does not stop other stations. The results are aggregated before the Basic LLM Chain calls the Ollama Chat Model. The prompt requires plain text under 150 words, one line per station, a practical non-alarmist step for at-risk stations, and a reminder that official warnings take priority. The Slack body is passed as a JSON object so model quotes and line breaks are safe.

## Testing

Run the workflow with **Execute Workflow** from the Manual Trigger. To exercise the alert branch without notifying Slack, temporarily set a low example threshold while leaving `slack_webhook_url` empty; the workflow will record the all-clear summary path because Slack is not configured. To test an actual notification, use a controlled Slack webhook and restore the intended threshold afterward.

## Error workflow

n8n templates cannot include an Error Trigger in the same workflow for this purpose. Create a separate workflow with an **Error Trigger**, add the desired notification or incident handling, and select that workflow under the main workflow's **Settings → Error Workflow**. The Open-Meteo request also continues on failure so one station without data does not discard the remaining stations.

## Honest limitations

This is decision support based on a hydrological model forecast, not an official flood-warning system, gauge network, or emergency service. Forecasts can be wrong, coordinates are mapped to the API's river grid, and the default thresholds are examples. Check local civil-protection and water-authority notices; official warnings always take priority. Ollama must be reachable from n8n, and the workflow requires outbound access to the Open-Meteo Flood API, which is free for non-commercial use (commercial use requires an Open-Meteo API plan).

## Español

Este flujo consulta pronósticos de caudal, marca puntos que podrían requerir atención y genera un boletín con un modelo Ollama local. Los umbrales incluidos son ejemplos: reemplázalos por los valores de la autoridad local. No es un sistema oficial de alertas: sigue siempre los avisos de Protección Civil y de la autoridad del agua (en México, CONAGUA).
