# TxBot Chatbot UI

A responsive web-based chatbot interface for the TxBot token management platform.

## Features

✨ **Modern Interface**
- Clean, gradient-based design
- Responsive mobile-friendly layout
- Smooth animations and transitions

💬 **Chat Functionality**
- Real-time message display
- User and bot message differentiation
- Automatic scrolling to latest messages
- Timestamp for each message

🎯 **Quick Start**
- No build tools required
- Pure HTML/CSS/JavaScript
- Easy to customize

## Files

- `index.html` - Main chatbot interface
- `styles.css` - Styling and animations
- `script.js` - Chat logic and message handling
- `README.md` - This file

## Usage

1. Open `index.html` in a web browser
2. Type a message in the input field
3. Press Enter or click Send
4. Bot responds based on keyword matching

## Customize

### Change Colors
Edit the gradient in `styles.css`:
```css
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
```

### Add Bot Responses
Edit `botResponses` in `script.js`:
```javascript
const botResponses = {
    'your-keyword': 'Your custom response',
    // ...
};
```

### Change Bot Name
Update `<title>` and `.chat-header h1` in `index.html`

## Integration

To connect to a real backend:

1. Replace the `getBotResponse()` function in `script.js` with an API call:
```javascript
async function getBotResponse(userMessage) {
    const response = await fetch('/api/chat', {
        method: 'POST',
        body: JSON.stringify({ message: userMessage })
    });
    const data = await response.json();
    return data.reply;
}
```

2. Update the chat form handler to use `await`

## Browser Support

- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

## License

MIT - Part of TxBot Project

## Next Steps

- [ ] Backend API integration
- [ ] User authentication
- [ ] Message persistence
- [ ] Admin dashboard
- [ ] Advanced AI responses
