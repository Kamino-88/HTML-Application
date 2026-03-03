<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Professional HTML Email Design Generator</title>
    <style>
        /* App Styles - Using modern CSS for the dashboard UI */
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }
        .dashboard {
            display: flex;
            height: 100vh;
        }
        .editor {
            width: 50%;
            padding: 20px;
            background-color: #fff;
            overflow-y: auto;
            border-right: 1px solid #ddd;
        }
        .preview {
            width: 50%;
            padding: 20px;
            background-color: #f4f4f4;
        }
        #preview-frame {
            width: 100%;
            height: 100%;
            border: 1px solid #ddd;
            background-color: #fff;
        }
        input, select {
            width: 100%;
            margin-bottom: 10px;
            padding: 8px;
            box-sizing: border-box;
        }
        .sections {
            display: flex;
            flex-wrap: wrap;
            margin-bottom: 20px;
        }
        .sections div {
            padding: 10px;
            background-color: #eee;
            margin: 5px;
            cursor: grab;
            user-select: none;
        }
        #body-editor {
            min-height: 300px;
            border: 1px dashed #ccc;
            padding: 10px;
            background-color: #fafafa;
        }
        .section {
            margin-bottom: 10px;
            padding: 10px;
            border: 1px solid #ddd;
            background-color: #fff;
            cursor: grab;
        }
        .section[contenteditable="true"] {
            min-height: 50px;
        }
        .section input {
            width: auto;
        }
        button {
            margin-right: 10px;
            padding: 10px 20px;
            background-color: #007bff;
            color: white;
            border: none;
            cursor: pointer;
        }
        button:hover {
            background-color: #0056b3;
        }
        /* Media queries for app responsiveness */
        @media (max-width: 768px) {
            .dashboard {
                flex-direction: column;
            }
            .editor, .preview {
                width: 100%;
                height: 50vh;
            }
        }
    </style>
</head>
<body>
    <div class="dashboard">
        <div class="editor">
            <h2>Email Editor</h2>
            
            <!-- Basic Inputs -->
            <input id="subject" type="text" placeholder="Email Subject">
            <input id="heading" type="text" placeholder="Email Heading">
            <input id="footer" type="text" placeholder="Footer Text">
            <input id="sender" type="text" placeholder="Sender Name">
            
            <!-- Design Options -->
            <h3>Design Customization</h3>
            <label>Theme Color: <input id="theme-color" type="color" value="#007bff"></label>
            <label>Font Style: 
                <select id="font-style">
                    <option value="Arial, sans-serif">Arial</option>
                    <option value="Helvetica, sans-serif">Helvetica</option>
                    <option value="Times New Roman, serif">Times New Roman</option>
                    <option value="Verdana, sans-serif">Verdana</option>
                    <option value="Georgia, serif">Georgia</option>
                </select>
            </label>
            <label>Header Background Color: <input id="header-bg" type="color" value="#f8f9fa"></label>
            <label>Button Color: <input id="button-color" type="color" value="#007bff"></label>
            <input id="button-text" type="text" placeholder="Default Button Text" value="Call to Action">
            
            <!-- Uploads -->
            <h3>Uploads</h3>
            <label>Logo: <input id="logo" type="file" accept="image/*"></label>
            <label>Banner: <input id="banner" type="file" accept="image/*"></label>
            
            <!-- Templates -->
            <h3>Select Template</h3>
            <select id="template">
                <option value="">Select Template</option>
                <option value="corporate">Corporate</option>
                <option value="minimal">Minimal</option>
                <option value="marketing">Marketing</option>
                <option value="institutional">Institutional</option>
            </select>
            
            <!-- Draggable Sections -->
            <h3>Drag-and-Drop Sections</h3>
            <div class="sections">
                <div draggable="true" data-type="text">Text</div>
                <div draggable="true" data-type="image">Image</div>
                <div draggable="true" data-type="button">Button</div>
                <div draggable="true" data-type="divider">Divider</div>
                <div draggable="true" data-type="social">Social Links</div>
            </div>
            
            <!-- Body Editor Dropzone -->
            <h3>Body Editor</h3>
            <div id="body-editor" class="dropzone"></div>
        </div>
        <div class="preview">
            <h2>Live Preview</h2>
            <iframe id="preview-frame"></iframe>
        </div>
    </div>
    
    <!-- Output Buttons -->
    <div style="padding: 20px; text-align: center;">
        <button onclick="copyHTML()">Copy HTML</button>
        <button onclick="downloadHTML()">Download .html</button>
        <button onclick="exportAll()">Export Subject + Body</button>
    </div>
    
    <script>
        // Global variables
        let logoData = '';
        let bannerData = '';
        let sectionCounter = 0;
        const bodyEditor = document.getElementById('body-editor');
        const previewFrame = document.getElementById('preview-frame');
        
        // Handle file uploads for logo and banner
        document.getElementById('logo').addEventListener('change', handleFileUpload('logo'));
        document.getElementById('banner').addEventListener('change', handleFileUpload('banner'));
        
        function handleFileUpload(type) {
            return function(event) {
                const file = event.target.files[0];
                if (file) {
                    const reader = new FileReader();
                    reader.onload = function(e) {
                        if (type === 'logo') logoData = e.target.result;
                        if (type === 'banner') bannerData = e.target.result;
                        updatePreview();
                    };
                    reader.readAsDataURL(file);
                }
            };
        }
        
        // Drag and Drop Setup
        const draggables = document.querySelectorAll('.sections div');
        draggables.forEach(draggable => {
            draggable.addEventListener('dragstart', dragStart);
        });
        
        bodyEditor.addEventListener('dragover', dragOver);
        bodyEditor.addEventListener('drop', drop);
        bodyEditor.addEventListener('dragenter', dragEnter);
        bodyEditor.addEventListener('dragleave', dragLeave);
        
        function dragStart(event) {
            event.dataTransfer.setData('type', event.target.dataset.type || 'existing');
            event.dataTransfer.setData('id', event.target.id || '');
        }
        
        function dragOver(event) {
            event.preventDefault();
            // Find the target element under the mouse
            let target = getDragAfterElement(bodyEditor, event.clientY);
            // Could add visual feedback here if needed
        }
        
        function dragEnter(event) {
            event.preventDefault();
            bodyEditor.style.borderColor = '#007bff';
        }
        
        function dragLeave() {
            bodyEditor.style.borderColor = '#ccc';
        }
        
        function drop(event) {
            event.preventDefault();
            bodyEditor.style.borderColor = '#ccc';
            const type = event.dataTransfer.getData('type');
            const id = event.dataTransfer.getData('id');
            let afterElement = getDragAfterElement(bodyEditor, event.clientY);
            
            if (type === 'existing' && id) {
                // Reorder existing section
                const section = document.getElementById(id);
                if (afterElement == null) {
                    bodyEditor.appendChild(section);
                } else {
                    bodyEditor.insertBefore(section, afterElement);
                }
            } else {
                // Add new section
                createSection(type, afterElement);
            }
            updatePreview();
        }
        
        // Helper to get the element after which to insert
        function getDragAfterElement(container, y) {
            const draggableElements = [...container.querySelectorAll('.section:not(.dragging)')];
            return draggableElements.reduce((closest, child) => {
                const box = child.getBoundingClientRect();
                const offset = y - box.top - box.height / 2;
                if (offset < 0 && offset > closest.offset) {
                    return { offset: offset, element: child };
                } else {
                    return closest;
                }
            }, { offset: Number.NEGATIVE_INFINITY }).element;
        }
        
        // Create new section based on type
        function createSection(type, insertBefore) {
            const section = document.createElement('div');
            section.classList.add('section', type);
            section.draggable = true;
            section.id = `section-${sectionCounter++}`;
            section.addEventListener('dragstart', dragStart);
            section.addEventListener('input', updatePreview); // For editable content
            
            switch (type) {
                case 'text':
                    section.contentEditable = true;
                    section.innerHTML = 'Enter your text here...';
                    break;
                case 'image':
                    const imgInput = document.createElement('input');
                    imgInput.type = 'file';
                    imgInput.accept = 'image/*';
                    imgInput.addEventListener('change', (e) => {
                        const file = e.target.files[0];
                        if (file) {
                            const reader = new FileReader();
                            reader.onload = (ev) => {
                                const img = section.querySelector('img') || document.createElement('img');
                                img.src = ev.target.result;
                                section.appendChild(img);
                                updatePreview();
                            };
                            reader.readAsDataURL(file);
                        }
                    });
                    section.appendChild(imgInput);
                    break;
                case 'button':
                    section.contentEditable = true;
                    section.innerHTML = document.getElementById('button-text').value;
                    const btnLink = document.createElement('input');
                    btnLink.type = 'text';
                    btnLink.placeholder = 'Button Link URL';
                    btnLink.addEventListener('input', updatePreview);
                    section.appendChild(btnLink);
                    break;
                case 'divider':
                    section.innerHTML = '<hr>';
                    break;
                case 'social':
                    const socialDiv = document.createElement('div');
                    socialDiv.innerHTML = `
                        <label>Facebook: <input type="text" placeholder="URL" class="social-fb"></label>
                        <label>Twitter: <input type="text" placeholder="URL" class="social-tw"></label>
                        <label>LinkedIn: <input type="text" placeholder="URL" class="social-li"></label>
                        <label>Instagram: <input type="text" placeholder="URL" class="social-ig"></label>
                    `;
                    socialDiv.addEventListener('input', updatePreview);
                    section.appendChild(socialDiv);
                    break;
            }
            
            if (insertBefore == null) {
                bodyEditor.appendChild(section);
            } else {
                bodyEditor.insertBefore(section, insertBefore);
            }
        }
        
        // Template Selection
        document.getElementById('template').addEventListener('change', (event) => {
            const value = event.target.value;
            bodyEditor.innerHTML = ''; // Clear existing
            sectionCounter = 0;
            
            switch (value) {
                case 'corporate':
                    document.getElementById('theme-color').value = '#004085';
                    document.getElementById('header-bg').value = '#e9ecef';
                    document.getElementById('button-color').value = '#007bff';
                    createSection('text');
                    createSection('button');
                    createSection('divider');
                    createSection('social');
                    break;
                case 'minimal':
                    document.getElementById('theme-color').value = '#6c757d';
                    document.getElementById('header-bg').value = '#ffffff';
                    document.getElementById('button-color').value = '#6c757d';
                    createSection('text');
                    break;
                case 'marketing':
                    document.getElementById('theme-color').value = '#28a745';
                    document.getElementById('header-bg').value = '#d4edda';
                    document.getElementById('button-color').value = '#28a745';
                    createSection('image');
                    createSection('text');
                    createSection('button');
                    break;
                case 'institutional':
                    document.getElementById('theme-color').value = '#343a40';
                    document.getElementById('header-bg').value = '#f8f9fa';
                    document.getElementById('button-color').value = '#343a40';
                    createSection('text');
                    createSection('divider');
                    createSection('text');
                    createSection('social');
                    break;
            }
            updatePreview();
        });
        
        // Add event listeners for all inputs to update preview
        const inputs = document.querySelectorAll('input, select');
        inputs.forEach(input => {
            if (input.type !== 'file') {
                input.addEventListener('input', updatePreview);
            }
        });
        
        // Generate Email HTML with Inline CSS
        function generateEmailHTML() {
            const subject = document.getElementById('subject').value;
            const heading = document.getElementById('heading').value;
            const footer = document.getElementById('footer').value;
            const sender = document.getElementById('sender').value;
            const font = document.getElementById('font-style').value;
            const themeColor = document.getElementById('theme-color').value;
            const headerBg = document.getElementById('header-bg').value;
            const buttonColor = document.getElementById('button-color').value;
            
            let bodyHTML = '';
            const sections = bodyEditor.querySelectorAll('.section');
            sections.forEach(section => {
                if (section.classList.contains('text')) {
                    bodyHTML += `<tr><td style="padding: 10px; font-family: ${font}; color: ${themeColor};">${section.innerHTML}</td></tr>`;
                } else if (section.classList.contains('image')) {
                    const img = section.querySelector('img');
                    if (img) {
                        bodyHTML += `<tr><td style="padding: 10px;"><img src="${img.src}" style="max-width: 100%; display: block;" alt="Image"></td></tr>`;
                    }
                } else if (section.classList.contains('button')) {
                    const text = section.innerText.trim();
                    const linkInput = section.querySelector('input');
                    const link = linkInput ? linkInput.value : '#';
                    bodyHTML += `<tr><td style="padding: 10px; text-align: center;"><a href="${link}" style="background-color: ${buttonColor}; color: #ffffff; padding: 10px 20px; text-decoration: none; border-radius: 5px; font-family: ${font};">${text || 'Button'}</a></td></tr>`;
                } else if (section.classList.contains('divider')) {
                    bodyHTML += `<tr><td style="padding: 10px 0;"><hr style="border: 0; height: 1px; background-color: ${themeColor};"></td></tr>`;
                } else if (section.classList.contains('social')) {
                    let socialLinks = '';
                    const fb = section.querySelector('.social-fb')?.value;
                    const tw = section.querySelector('.social-tw')?.value;
                    const li = section.querySelector('.social-li')?.value;
                    const ig = section.querySelector('.social-ig')?.value;
                    if (fb) socialLinks += `<a href="${fb}" style="color: ${themeColor}; text-decoration: none; padding: 0 5px;">Facebook</a>`;
                    if (tw) socialLinks += `<a href="${tw}" style="color: ${themeColor}; text-decoration: none; padding: 0 5px;">Twitter</a>`;
                    if (li) socialLinks += `<a href="${li}" style="color: ${themeColor}; text-decoration: none; padding: 0 5px;">LinkedIn</a>`;
                    if (ig) socialLinks += `<a href="${ig}" style="color: ${themeColor}; text-decoration: none; padding: 0 5px;">Instagram</a>`;
                    if (socialLinks) {
                        bodyHTML += `<tr><td style="padding: 10px; text-align: center; font-family: ${font};">${socialLinks}</td></tr>`;
                    }
                }
            });
            
            const html = `
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>${subject}</title>
</head>
<body style="margin: 0; padding: 0; background-color: #f4f4f4;">
    <table border="0" cellpadding="0" cellspacing="0" width="100%" style="max-width: 600px; margin: 0 auto; background-color: #ffffff; font-family: ${font};">
        <!-- Header -->
        <tr>
            <td style="background-color: ${headerBg}; padding: 20px; text-align: center;">
                ${logoData ? `<img src="${logoData}" alt="Logo" style="max-height: 50px; display: block; margin: 0 auto;">` : ''}
                ${bannerData ? `<img src="${bannerData}" alt="Banner" style="width: 100%; display: block; margin-top: 10px;">` : ''}
            </td>
        </tr>
        <!-- Heading -->
        <tr>
            <td style="padding: 20px; text-align: center; font-size: 24px; color: ${themeColor};">
                ${heading}
            </td>
        </tr>
        <!-- Body Sections -->
        ${bodyHTML}
        <!-- Footer -->
        <tr>
            <td style="padding: 20px; text-align: center; font-size: 12px; color: #6c757d;">
                ${footer}
            </td>
        </tr>
        <tr>
            <td style="padding: 0 20px 20px; text-align: center; font-size: 12px; color: #6c757d;">
                Sent by ${sender}
            </td>
        </tr>
    </table>
</body>
</html>
            `;
            return html;
        }
        
        // Update Live Preview
        function updatePreview() {
            const html = generateEmailHTML();
            previewFrame.srcdoc = html;
        }
        
        // Copy HTML to Clipboard
        function copyHTML() {
            const html = generateEmailHTML();
            navigator.clipboard.writeText(html).then(() => {
                alert('HTML copied to clipboard!');
            });
        }
        
        // Download as .html
        function downloadHTML() {
            const html = generateEmailHTML();
            const blob = new Blob([html], { type: 'text/html' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'email.html';
            a.click();
            URL.revokeObjectURL(url);
        }
        
        // Export Subject + Body (as JSON for simplicity)
        function exportAll() {
            const subject = document.getElementById('subject').value;
            const html = generateEmailHTML();
            const data = { subject, html };
            const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'email-export.json';
            a.click();
            URL.revokeObjectURL(url);
        }
        
        // Initial preview update
        updatePreview();
    </script>
</body>
</html>
