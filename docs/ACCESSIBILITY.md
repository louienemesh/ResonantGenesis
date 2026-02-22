# Accessibility Guide

Complete guide to accessibility features, WCAG compliance, and assistive technology support in ResonantGenesis.

## Overview

ResonantGenesis is committed to providing an accessible platform for all users. This guide covers accessibility features, compliance standards, and best practices for building accessible agents.

## Compliance Standards

### WCAG 2.1 Compliance

| Level | Status | Coverage |
|-------|--------|----------|
| Level A | ✅ Compliant | 100% |
| Level AA | ✅ Compliant | 100% |
| Level AAA | 🔄 Partial | 75% |

### Section 508

The platform complies with Section 508 requirements for federal accessibility standards.

### ADA Compliance

ResonantGenesis meets ADA (Americans with Disabilities Act) requirements for digital accessibility.

## Accessibility Features

### Visual Accessibility

| Feature | Description |
|---------|-------------|
| High Contrast Mode | Enhanced color contrast |
| Dark Mode | Reduced eye strain |
| Font Scaling | Adjustable text size |
| Color Blindness Support | Deuteranopia, protanopia, tritanopia modes |
| Focus Indicators | Visible focus states |

### Motor Accessibility

| Feature | Description |
|---------|-------------|
| Keyboard Navigation | Full keyboard support |
| Skip Links | Skip to main content |
| Large Click Targets | Minimum 44x44px targets |
| Reduced Motion | Disable animations |
| Voice Control | Voice command support |

### Auditory Accessibility

| Feature | Description |
|---------|-------------|
| Captions | Video captions |
| Transcripts | Audio transcripts |
| Visual Alerts | Visual notification alternatives |
| Sign Language | Video sign language (planned) |

### Cognitive Accessibility

| Feature | Description |
|---------|-------------|
| Simple Language | Clear, concise content |
| Consistent Navigation | Predictable layouts |
| Error Prevention | Confirmation dialogs |
| Help Text | Contextual assistance |
| Reading Level | Grade 8 reading level |

## Configuration

### Enable Accessibility Features

```python
# Configure accessibility preferences
client.users.update_preferences(
    accessibility={
        "high_contrast": True,
        "reduced_motion": True,
        "font_scale": 1.25,
        "screen_reader_optimized": True
    }
)
```

### Accessibility Settings

```python
# Get current accessibility settings
settings = client.accessibility.get_settings()

print(f"High contrast: {settings.high_contrast}")
print(f"Font scale: {settings.font_scale}")
print(f"Reduced motion: {settings.reduced_motion}")
```

## Keyboard Navigation

### Global Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` | Move to next element |
| `Shift + Tab` | Move to previous element |
| `Enter` | Activate element |
| `Escape` | Close modal/menu |
| `Space` | Toggle checkbox/button |
| `Arrow keys` | Navigate within components |

### Application Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl/Cmd + K` | Open command palette |
| `Ctrl/Cmd + /` | Open keyboard shortcuts |
| `Ctrl/Cmd + S` | Save current work |
| `Ctrl/Cmd + N` | Create new agent |
| `Ctrl/Cmd + E` | Execute agent |

### Skip Links

```html
<!-- Skip to main content -->
<a href="#main-content" class="skip-link">
  Skip to main content
</a>

<!-- Skip to navigation -->
<a href="#navigation" class="skip-link">
  Skip to navigation
</a>
```

## Screen Reader Support

### Supported Screen Readers

| Screen Reader | Platform | Status |
|---------------|----------|--------|
| NVDA | Windows | ✅ Full |
| JAWS | Windows | ✅ Full |
| VoiceOver | macOS/iOS | ✅ Full |
| TalkBack | Android | ✅ Full |
| Narrator | Windows | ✅ Full |
| Orca | Linux | ✅ Full |

### ARIA Labels

```python
# Configure agent with accessible labels
agent = client.agents.create(
    name="Support Assistant",
    accessibility={
        "aria_label": "Customer support chat assistant",
        "aria_description": "An AI assistant that helps with customer inquiries",
        "role": "application"
    }
)
```

### Live Regions

```python
# Configure live region announcements
client.accessibility.configure_live_regions({
    "session_updates": "polite",
    "error_messages": "assertive",
    "status_changes": "polite"
})
```

## Color and Contrast

### Color Contrast Ratios

| Element | Minimum Ratio | Current |
|---------|---------------|---------|
| Normal text | 4.5:1 | 7.2:1 |
| Large text | 3:1 | 5.8:1 |
| UI components | 3:1 | 4.5:1 |
| Focus indicators | 3:1 | 4.5:1 |

### Color Blindness Modes

```python
# Enable color blindness mode
client.accessibility.set_color_mode("deuteranopia")

# Available modes:
# - "normal"
# - "deuteranopia" (green-blind)
# - "protanopia" (red-blind)
# - "tritanopia" (blue-blind)
# - "achromatopsia" (monochrome)
```

### High Contrast Theme

```python
# Enable high contrast mode
client.accessibility.enable_high_contrast(True)

# Configure custom high contrast colors
client.accessibility.configure_high_contrast({
    "background": "#000000",
    "foreground": "#FFFFFF",
    "accent": "#FFFF00",
    "error": "#FF0000",
    "success": "#00FF00"
})
```

## Text and Typography

### Font Scaling

```python
# Set font scale
client.accessibility.set_font_scale(1.5)  # 150% of default

# Available scales: 0.75, 1.0, 1.25, 1.5, 1.75, 2.0
```

### Dyslexia-Friendly Mode

```python
# Enable dyslexia-friendly mode
client.accessibility.enable_dyslexia_mode(True)

# Features:
# - OpenDyslexic font
# - Increased letter spacing
# - Increased line height
# - Reduced text density
```

### Reading Assistance

```python
# Enable reading assistance
client.accessibility.configure_reading({
    "line_focus": True,
    "word_spacing": "wide",
    "paragraph_spacing": "wide",
    "text_to_speech": True
})
```

## Motion and Animation

### Reduced Motion

```python
# Enable reduced motion
client.accessibility.set_reduced_motion(True)

# Disables:
# - Animations
# - Transitions
# - Auto-playing videos
# - Parallax effects
```

### Animation Preferences

```python
# Configure animation preferences
client.accessibility.configure_animations({
    "enabled": True,
    "duration_multiplier": 2.0,  # Slower animations
    "disable_parallax": True,
    "disable_auto_play": True
})
```

## Accessible Agents

### Building Accessible Agents

```python
# Create accessible agent
agent = client.agents.create(
    name="Accessible Assistant",
    system_prompt="""You are an accessible assistant.
    
    Guidelines:
    - Use clear, simple language
    - Avoid jargon and idioms
    - Provide text alternatives for visual content
    - Structure responses with headings
    - Use lists for multiple items
    - Confirm understanding when needed
    """,
    accessibility={
        "plain_language": True,
        "reading_level": "grade_8",
        "structure_responses": True
    }
)
```

### Response Formatting

```python
# Configure accessible response formatting
agent = client.agents.create(
    name="Structured Assistant",
    config={
        "response_format": {
            "use_headings": True,
            "use_lists": True,
            "max_paragraph_length": 3,
            "include_summaries": True
        }
    }
)
```

### Alternative Text

```python
# Generate alt text for images
alt_text = client.accessibility.generate_alt_text(
    image_url="https://example.com/chart.png"
)

print(f"Alt text: {alt_text.description}")
# "Bar chart showing monthly sales from January to December 2026"
```

## Testing Accessibility

### Automated Testing

```python
# Run accessibility audit
audit = client.accessibility.run_audit(
    url="https://app.resonantgenesis.xyz/dashboard"
)

print(f"Score: {audit.score}/100")
print(f"Issues: {audit.issue_count}")

for issue in audit.issues:
    print(f"[{issue.severity}] {issue.description}")
    print(f"  Element: {issue.selector}")
    print(f"  Fix: {issue.recommendation}")
```

### Manual Testing Checklist

```markdown
## Keyboard Navigation
- [ ] All interactive elements are focusable
- [ ] Focus order is logical
- [ ] Focus is visible
- [ ] No keyboard traps

## Screen Reader
- [ ] All images have alt text
- [ ] Form fields have labels
- [ ] Headings are hierarchical
- [ ] Links are descriptive

## Visual
- [ ] Color contrast meets WCAG AA
- [ ] Information not conveyed by color alone
- [ ] Text is resizable to 200%
- [ ] No horizontal scrolling at 320px

## Forms
- [ ] Error messages are clear
- [ ] Required fields are indicated
- [ ] Instructions are provided
- [ ] Validation is accessible
```

### Accessibility Report

```python
# Generate accessibility report
report = client.accessibility.generate_report(
    period="30d",
    include_recommendations=True
)

print(f"Overall score: {report.score}")
print(f"Improvement: {report.improvement}%")

for category in report.categories:
    print(f"{category.name}: {category.score}/100")
```

## Assistive Technology Integration

### Voice Control

```python
# Enable voice control
client.accessibility.enable_voice_control({
    "enabled": True,
    "wake_word": "hey assistant",
    "commands": {
        "create agent": "agents.create",
        "run session": "sessions.execute",
        "go to dashboard": "navigate.dashboard"
    }
})
```

### Switch Control

```python
# Configure switch control
client.accessibility.configure_switch_control({
    "enabled": True,
    "scan_speed": 2.0,  # seconds
    "auto_scan": True,
    "highlight_color": "#FFFF00"
})
```

### Eye Tracking

```python
# Enable eye tracking support
client.accessibility.enable_eye_tracking({
    "enabled": True,
    "dwell_time": 1.0,  # seconds
    "calibration_required": True
})
```

## Documentation Accessibility

### Accessible Documentation

All documentation follows accessibility guidelines:

- **Headings**: Proper heading hierarchy
- **Links**: Descriptive link text
- **Images**: Alt text for all images
- **Code**: Syntax highlighting with contrast
- **Tables**: Proper table headers
- **Lists**: Semantic list markup

### Alternative Formats

| Format | Availability |
|--------|--------------|
| HTML | ✅ Available |
| PDF (accessible) | ✅ Available |
| EPUB | ✅ Available |
| Plain text | ✅ Available |
| Audio | 🔄 Coming soon |
| Braille | 📋 On request |

## Best Practices

### For Developers

1. **Use semantic HTML** - Proper element usage
2. **Add ARIA labels** - When HTML is insufficient
3. **Test with screen readers** - Regular testing
4. **Keyboard test** - Navigate without mouse
5. **Check contrast** - Use contrast checkers

### For Content

1. **Write clearly** - Simple, direct language
2. **Structure content** - Use headings and lists
3. **Describe images** - Meaningful alt text
4. **Link descriptively** - Avoid "click here"
5. **Provide alternatives** - Multiple formats

### For Agents

1. **Plain language** - Avoid jargon
2. **Confirm understanding** - Check comprehension
3. **Offer alternatives** - Different explanation styles
4. **Be patient** - Allow time for responses
5. **Provide summaries** - Recap key points

## Reporting Issues

### Report Accessibility Issue

```python
# Report accessibility issue
issue = client.accessibility.report_issue(
    description="Button not reachable via keyboard",
    page_url="https://app.resonantgenesis.xyz/agents",
    assistive_technology="NVDA",
    severity="high"
)

print(f"Issue ID: {issue.id}")
print(f"Status: {issue.status}")
```

### Contact

- **Accessibility Team**: accessibility@resonantgenesis.xyz
- **Phone**: +1-XXX-XXX-XXXX (TTY available)
- **Response Time**: 24-48 hours

## API Reference

### Get Settings

```bash
GET /api/v1/accessibility/settings
```

### Update Settings

```bash
PATCH /api/v1/accessibility/settings
```

### Run Audit

```bash
POST /api/v1/accessibility/audit
```

### Generate Alt Text

```bash
POST /api/v1/accessibility/alt-text
```

### Report Issue

```bash
POST /api/v1/accessibility/issues
```

---

**Need accessibility help?** Contact accessibility@resonantgenesis.xyz or call our accessibility hotline.
