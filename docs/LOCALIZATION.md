# Localization Guide

Complete guide to internationalization, language support, and localization features in ResonantGenesis.

## Overview

ResonantGenesis supports multiple languages and locales to serve a global user base. This guide covers language configuration, translation management, and best practices for building multilingual agents.

## Supported Languages

### Platform Languages

| Language | Code | Status |
|----------|------|--------|
| English | en | ✅ Full |
| Spanish | es | ✅ Full |
| French | fr | ✅ Full |
| German | de | ✅ Full |
| Portuguese | pt | ✅ Full |
| Japanese | ja | ✅ Full |
| Chinese (Simplified) | zh-CN | ✅ Full |
| Chinese (Traditional) | zh-TW | ✅ Full |
| Korean | ko | ✅ Full |
| Italian | it | 🔄 Partial |
| Dutch | nl | 🔄 Partial |
| Russian | ru | 🔄 Partial |
| Arabic | ar | 🔄 Partial |
| Hindi | hi | 📋 Planned |

### Agent Languages

Agents can communicate in any language supported by the underlying model. Most models support 50+ languages.

## Language Configuration

### User Language

```python
# Set user language preference
client.users.update_preferences(
    language="es",
    locale="es-MX",
    timezone="America/Mexico_City"
)
```

### Organization Language

```python
# Set organization default language
client.organizations.update_settings(
    org_id="org_123",
    settings={
        "default_language": "fr",
        "default_locale": "fr-FR",
        "allowed_languages": ["en", "fr", "de", "es"]
    }
)
```

### Agent Language

```python
# Create multilingual agent
agent = client.agents.create(
    name="Support Assistant",
    system_prompt="""You are a multilingual support assistant.
    Respond in the same language the user writes in.
    If unsure, ask for language preference.""",
    config={
        "default_language": "en",
        "supported_languages": ["en", "es", "fr", "de"],
        "auto_detect_language": True
    }
)
```

## Language Detection

### Automatic Detection

```python
# Enable automatic language detection
agent = client.agents.create(
    name="Global Assistant",
    config={
        "auto_detect_language": True,
        "language_detection_confidence": 0.8,
        "fallback_language": "en"
    }
)
```

### Manual Detection

```python
# Detect language of text
detection = client.localization.detect_language(
    text="Bonjour, comment puis-je vous aider?"
)

print(f"Language: {detection.language}")  # fr
print(f"Confidence: {detection.confidence}")  # 0.98
print(f"Script: {detection.script}")  # Latin
```

## Translation

### Translate Text

```python
# Translate text
translation = client.localization.translate(
    text="Hello, how can I help you?",
    source_language="en",
    target_language="es"
)

print(f"Translation: {translation.text}")
# "Hola, ¿cómo puedo ayudarte?"
```

### Batch Translation

```python
# Translate multiple texts
translations = client.localization.translate_batch(
    texts=[
        "Welcome to our platform",
        "Create your first agent",
        "View documentation"
    ],
    source_language="en",
    target_language="ja"
)

for t in translations:
    print(f"{t.original} -> {t.translated}")
```

### Translation Memory

```python
# Use translation memory for consistency
translation = client.localization.translate(
    text="Agent",
    source_language="en",
    target_language="de",
    use_translation_memory=True,
    context="AI agent software"
)

# Returns consistent translation based on previous translations
```

## Locale Settings

### Date and Time

```python
# Configure locale-specific formatting
client.localization.configure_locale(
    locale="de-DE",
    settings={
        "date_format": "DD.MM.YYYY",
        "time_format": "HH:mm",
        "first_day_of_week": "monday",
        "timezone": "Europe/Berlin"
    }
)
```

### Number Formatting

```python
# Format numbers by locale
formatted = client.localization.format_number(
    number=1234567.89,
    locale="de-DE"
)
print(formatted)  # "1.234.567,89"

formatted = client.localization.format_number(
    number=1234567.89,
    locale="en-US"
)
print(formatted)  # "1,234,567.89"
```

### Currency Formatting

```python
# Format currency by locale
formatted = client.localization.format_currency(
    amount=99.99,
    currency="EUR",
    locale="de-DE"
)
print(formatted)  # "99,99 €"

formatted = client.localization.format_currency(
    amount=99.99,
    currency="USD",
    locale="en-US"
)
print(formatted)  # "$99.99"
```

## Content Localization

### Localized Prompts

```python
# Create agent with localized prompts
agent = client.agents.create(
    name="Customer Support",
    localized_prompts={
        "en": "You are a helpful customer support agent.",
        "es": "Eres un agente de soporte al cliente útil.",
        "fr": "Vous êtes un agent de support client serviable.",
        "de": "Sie sind ein hilfreicher Kundensupport-Agent."
    }
)
```

### Localized Responses

```python
# Configure localized response templates
templates = client.localization.create_templates(
    name="greeting",
    translations={
        "en": "Hello! How can I help you today?",
        "es": "¡Hola! ¿Cómo puedo ayudarte hoy?",
        "fr": "Bonjour! Comment puis-je vous aider aujourd'hui?",
        "de": "Hallo! Wie kann ich Ihnen heute helfen?"
    }
)
```

### Dynamic Content

```python
# Use placeholders in translations
template = client.localization.create_template(
    name="welcome_user",
    translations={
        "en": "Welcome, {name}! You have {count} notifications.",
        "es": "¡Bienvenido, {name}! Tienes {count} notificaciones.",
        "fr": "Bienvenue, {name}! Vous avez {count} notifications."
    }
)

# Render with variables
message = client.localization.render_template(
    template_id=template.id,
    language="es",
    variables={"name": "Carlos", "count": 5}
)
# "¡Bienvenido, Carlos! Tienes 5 notificaciones."
```

## RTL Support

### Right-to-Left Languages

```python
# Configure RTL support
client.localization.configure_rtl(
    enabled=True,
    languages=["ar", "he", "fa", "ur"]
)
```

### Text Direction

```python
# Get text direction for language
direction = client.localization.get_text_direction("ar")
print(direction)  # "rtl"

direction = client.localization.get_text_direction("en")
print(direction)  # "ltr"
```

## Translation Management

### Export Translations

```python
# Export translations for external editing
export = client.localization.export_translations(
    languages=["en", "es", "fr"],
    format="json"
)

print(f"Download URL: {export.download_url}")
```

### Import Translations

```python
# Import translations from file
import_result = client.localization.import_translations(
    file_path="/path/to/translations.json",
    language="es",
    merge_strategy="overwrite"  # overwrite, skip, merge
)

print(f"Imported: {import_result.imported_count}")
print(f"Skipped: {import_result.skipped_count}")
```

### Translation Status

```python
# Get translation coverage
coverage = client.localization.get_coverage()

for lang in coverage.languages:
    print(f"{lang.code}: {lang.coverage}% complete")
    print(f"  Missing: {lang.missing_count} strings")
```

## Multilingual Agents

### Language Routing

```python
# Route to language-specific agents
team = client.teams.create(
    name="Global Support Team",
    routing={
        "type": "language",
        "agents": {
            "en": "agent_english",
            "es": "agent_spanish",
            "fr": "agent_french",
            "default": "agent_english"
        }
    }
)
```

### Language Handoff

```python
# Configure language handoff
agent = client.agents.create(
    name="Multilingual Router",
    config={
        "language_handoff": {
            "enabled": True,
            "threshold": 0.7,
            "handoff_message": {
                "en": "Transferring you to a specialist...",
                "es": "Transfiriéndote a un especialista...",
                "fr": "Transfert vers un spécialiste..."
            }
        }
    }
)
```

## API Localization

### Localized Errors

```python
# Get localized error messages
try:
    result = client.agents.get("invalid_id")
except APIError as e:
    # Error message in user's language
    print(e.localized_message)
```

### Localized Responses

```python
# Request localized API responses
client = ResonantGenesis(
    api_key="your_key",
    language="es"
)

# All API responses will include Spanish translations
```

## Best Practices

### For Developers

1. **Use placeholders** - Never concatenate translated strings
2. **Provide context** - Help translators understand usage
3. **Test RTL** - Verify layout with RTL languages
4. **Handle plurals** - Use proper plural forms
5. **Avoid hardcoding** - Externalize all strings

### For Content

1. **Keep it simple** - Simple text translates better
2. **Avoid idioms** - They don't translate well
3. **Be consistent** - Use same terms throughout
4. **Provide glossary** - Define key terms
5. **Review translations** - Native speaker review

### For Agents

1. **Detect language** - Auto-detect user language
2. **Confirm language** - Ask if unsure
3. **Stay consistent** - Don't switch languages mid-conversation
4. **Handle mixed** - Support code-switching
5. **Respect preferences** - Honor user settings

## Pluralization

### Plural Rules

```python
# Configure plural forms
template = client.localization.create_template(
    name="item_count",
    translations={
        "en": {
            "one": "You have {count} item",
            "other": "You have {count} items"
        },
        "ru": {
            "one": "У вас {count} элемент",
            "few": "У вас {count} элемента",
            "many": "У вас {count} элементов",
            "other": "У вас {count} элемента"
        }
    }
)

# Render with count
message = client.localization.render_plural(
    template_id=template.id,
    language="ru",
    count=5
)
# "У вас 5 элементов"
```

## Glossary

### Create Glossary

```python
# Create translation glossary
glossary = client.localization.create_glossary(
    name="Technical Terms",
    entries=[
        {"term": "Agent", "translation": "Agente", "language": "es"},
        {"term": "Session", "translation": "Sesión", "language": "es"},
        {"term": "Webhook", "translation": "Webhook", "language": "es"}
    ]
)
```

### Use Glossary

```python
# Translate using glossary
translation = client.localization.translate(
    text="Create a new Agent and configure the Webhook",
    target_language="es",
    glossary_id=glossary.id
)
# Uses glossary terms for consistency
```

## API Reference

### Detect Language

```bash
POST /api/v1/localization/detect
```

### Translate Text

```bash
POST /api/v1/localization/translate
```

### Get Supported Languages

```bash
GET /api/v1/localization/languages
```

### Export Translations

```bash
GET /api/v1/localization/export
```

### Import Translations

```bash
POST /api/v1/localization/import
```

### Get Coverage

```bash
GET /api/v1/localization/coverage
```

---

**Need localization help?** Contact localization@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
