
```markdown
# Setting Up Client-Side Files

This document outlines the steps to set up the client-side files required for generating and displaying PDF invoices using your web application.

## Step 1: Create the Directory Structure

Create the following directory structure in your client project:

```
/your-client-project
|-- /components
|   |-- PDFViewer.js
|-- /pages
|   |-- invoice.js
|-- /styles
|   |-- PDFViewer.module.css
```

### 1. Create the Directories

You can create the directories using the following commands:

```bash
mkdir -p components pages styles
```

## Step 2: Install Dependencies

Ensure you have the following dependencies installed in your client project:

```bash
npm install axios jsPDF jspdf-autotable
```

## Step 3: Create and Configure Files

### 1. Create `PDFViewer.js`

Create a file named `PDFViewer.js` in the `components` directory and add the following code:

```javascript
import React from 'react';
import styles from '../styles/PDFViewer.module.css';

const PDFViewer = ({ pdfUrl }) => {
  return (
    <div className={styles.container}>
      <iframe
        src={pdfUrl}
        width="100%"
        height="600"
        frameBorder="0"
        title="PDF Invoice"
      />
      <div className={styles.actions}>
        <a href={pdfUrl} download>
          <button>Download</button>
        </a>
        <button onClick={() => window.print()}>Print</button>
      </div>
    </div>
  );
};

export default PDFViewer;
```

### 2. Create `invoice.js`

Create a file named `invoice.js` in the `pages` directory and add the following code:

```javascript
import React, { useEffect, useState } from 'react';
import axios from 'axios';
import PDFViewer from '../components/PDFViewer';

const InvoicePage = ({ billNo }) => {
  const [pdfUrl, setPdfUrl] = useState('');

  useEffect(() => {
    const fetchPDF = async () => {
      try {
        const response = await axios.get(`/api/pdf/invoice/${billNo}`);
        setPdfUrl(response.data.url);
      } catch (error) {
        console.error('Error fetching PDF:', error);
      }
    };

    fetchPDF();
  }, [billNo]);

  if (!pdfUrl) {
    return <div>Loading...</div>;
  }

  return (
    <div>
      <h1>Invoice</h1>
      <PDFViewer pdfUrl={pdfUrl} />
    </div>
  );
};

// This will be called at build time
export async function getServerSideProps(context) {
  const { billNo } = context.params;

  return {
    props: { billNo }, // will be passed to the page component as props
  };
}

export default InvoicePage;
```

### 3. Create `PDFViewer.module.css`

Create a file named `PDFViewer.module.css` in the `styles` directory and add the following styles:

```css
.container {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.actions {
  margin-top: 20px;
  display: flex;
  justify-content: space-between;
  width: 100%;
}

button {
  background-color: #0070f3;
  color: white;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  margin: 0 10px;
  border-radius: 5px;
}

button:hover {
  background-color: #005bb5;
}
```

## Conclusion

You have successfully set up the client-side files required for generating and displaying PDF invoices in your web application. The `InvoicePage` component will fetch the PDF from your server and render it using the `PDFViewer` component.
