---
title: "Creating a Dynamic Userbox Generator with React"
excerpt: "A small project that helped me learn how to create interactive React-based web applications. <br/><img src='/images/userboxgenerator.png'>"
collection: portfolio
---

A **userbox** is a small, stylized graphic often used in online communities (forums, wikis, blogs, you name it) to display user information, affiliations, or personality traits. Traditionally, creating these requires manual image editing—a tedious process for non-designers.  

To practice building interactive web applications and solve the tedious nature of manually creating userbox graphics, I created a **Userbox Generator** as a React-based web app that lets users:  
✅ **Customize text, colors, and images** in real-time.  
✅ **Preview changes instantly** before exporting.  
✅ **Download the final design** as a PNG.  

!(/images/userboxgenerator.png)

Below, I’ll break down the technical implementation, design choices, and lessons learned.  

---

## **Step 1: Setting Up the React App**  

### **Why React?**  
React’s **component-based architecture** makes it ideal for dynamic UIs. In this case, the app’s core functionality revolves around:  
- **State management** (tracking user inputs like text/colors).  
- **Real-time previews** (updating the userbox on every change).  
- **Image handling** (uploading/displaying user-provided images).  

I initialized the project using `create-react-app`:  

```bash
npx create-react-app userbox-generator
cd userbox-generator
npm install react-color html2canvas
```

### **Key Dependencies**  
- **`react-color`**: A color picker component for selecting background/text/border colors.  
- **`html2canvas`**: Converts the userbox HTML element into a downloadable PNG.  

---

## **Step 2: Building the User Interface**  

The UI consists of:  
1. **Preview Section** (Live userbox display).  
2. **Controls Section** (Text, font size, color pickers, image upload).  

### **Preview Section (`userboxRef`)**  
The live preview is a `<div>` styled dynamically via React state:  

<!-- {% raw %} -->
```javascript
<div className="userbox-preview" ref={userboxRef}>
  <div
    className="userbox"
    style={{
      backgroundColor,
      border: `2px solid ${borderColor}`,
    }}
  >
    {/* Image and text components */}
  </div>
</div>
```
<!-- {% endraw %} -->

- **`useRef`** hooks capture the DOM element for `html2canvas` to convert it to an image later.  
- **Inline styles** update instantly when state changes (e.g., `backgroundColor`).  

### **Controls Section**  
Users adjust:  
- **Text content** (`<input type="text">` bound to `text` state).  
- **Font size** (`<input type="number">` bound to `fontSize` state).  
- **Colors** (`react-color` pickers for background/text/border).  

<!-- {% raw %} -->
```javascript
<SketchPicker 
  color={backgroundColor} 
  onChange={(color) => setBackgroundColor(color.hex)} 
/>
```
<!-- {% endraw %} -->

---

## **Step 3: Handling User Inputs**  

### **Text and Font Size**  
Basic input fields update state on change:  

```javascript
<input 
  type="text" 
  value={text} 
  onChange={(e) => setText(e.target.value)} 
/>
```

### **Image Uploads**  
Users can:  
1. **Upload a file** (handled via `URL.createObjectURL`).  
2. **Load from a URL** (fetched when clicking "Load Image").  

```javascript
const handleImageUpload = (e) => {
  const file = e.target.files[0];
  if (file) {
    setUserImage(URL.createObjectURL(file));
  }
};

const handleLoadImage = () => {
  setUserImage(imageUrl); // `imageUrl` is from a text input
};
```

---

## **Step 4: Generating the Downloadable PNG**  

The **`html2canvas`** library captures the `userbox-preview` div and converts it to a PNG:  

```javascript
const handleDownload = () => {
  html2canvas(userboxRef.current).then((canvas) => {
    const link = document.createElement('a');
    link.href = canvas.toDataURL('image/png');
    link.download = 'userbox.png';
    link.click();
  });
};
```

**How it works:**  
1. `html2canvas` renders the DOM element into a canvas.  
2. The canvas is converted to a data URL (base64 PNG).  
3. A temporary `<a>` tag triggers the download.  

---

## **Step 5: Styling with CSS**  

The app uses **Flexbox** for a responsive layout:  

```css
.userbox-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.color-section {
  display: flex;
  gap: 10px; /* Spacing between color pickers */
}
```

**Key design choices:**  
- **Centered layout** for focus on the userbox.  
- **Consistent spacing** between controls.  
- **Mobile-friendly** (scales down for smaller screens).  

---

## **Challenges and Solutions**  

### **1. Image Cross-Origin Issues**  
- **Problem:** `html2canvas` fails if images are hosted externally (CORS restrictions).  
- **Fix:** Added `useCORS: true` to `html2canvas` options.  

### **2. Dynamic Sizing**  
- **Problem:** Long text overflows the userbox.  
- **Fix:** Added `word-wrap: break-word` and `max-width` constraints.  

### **3. State Management**  
- **Problem:** Too many state variables (text, colors, image).  
- **Fix:** Combined related states into a single reducer (future optimization).  

---

## **Future Improvements**  
- Add **templates** (pre-designed userbox styles).  
- Support **social media sharing** (directly post to a social media platform of your choice).  
- Implement **user accounts** to save and share designs. 
- Implement a tagging system to query shared designs in a database.

---

**Try it yourself:**  
```bash
gh repo clone girubato/userboxGenerator
```
[GitHub Repo](https://github.com/girubato/userboxGenerator)
