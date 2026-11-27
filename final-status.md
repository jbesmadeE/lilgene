# Extension Status

## LinkedIn Compose Trigger
The trigger infrastructure is in place:
- **manifest.json**: content_scripts matches `https://www.linkedin.com/sharing/compose*`
- **content.js**: Detects LinkedIn URL, sets `state.mode = "linkedin"`
- **background.js**: `generateLinkedInPost()` function using SKILL.MD LinkedIn rules
- **SKILL.MD**: LinkedIn rules at the top specifying exact post content

## Current code/js/content.js state (lines 85-86):
```javascript
const panelHost = null;
const launcherHost = null;
```

## To use:
1. Load extension in Chrome: `chrome://extensions` → Developer mode → Load unpacked → `D:\AI job helper`
2. Visit: `https://www.linkedin.com/sharing/compose/`
3. Panel should appear and auto-generate job-seeking post

## Note
The AI post generation requires an AI provider key configured in the extension options. The post will use the rules from SKILL.MD with your portfolio.md data.