
# Setting Up Server Files

This document provides step-by-step instructions for setting up the server files required for generating PDF invoices and managing uploads to Amazon S3.

## Step 1: Create the Directory Structure

Create the following directory structure in your project:

```
/your-project
|-- /server
|   |-- AWSconfig.js
|   |-- /lib
|   |   |-- downloadPDF.js
|   |   |-- generatePDF.js
|   |-- /services
|   |   |-- s3Service.js
|   |-- PDFRoutes.js
|   |-- index.js
```

### 1. Create the Directories

You can create the directories using the following commands:

```bash
mkdir -p server/lib server/services
```

## Step 2: Create and Configure Files

### 1. Create `AWSconfig.js`

Create a file named `AWSconfig.js` in the `server` directory and add the following code:

```javascript
import AWS from 'aws-sdk';
import dotenv from 'dotenv';

dotenv.config();

AWS.config.update({
  accessKeyId: process.env.AWS_ACCESS_KEY_ID,
  secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  region: process.env.AWS_REGION,
});

const s3 = new AWS.S3();

export default s3;
```

### 2. Create `s3Service.js`

Create a file named `s3Service.js` in the `server/services` directory and add the following code:

```javascript
import { S3Client, GetObjectCommand, PutObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
import dotenv from "dotenv";

dotenv.config();

const s3Client = new S3Client({ region: process.env.AWS_REGION });

export async function uploadPDFToS3(buffer, fileName) {
  const uploadParams = {
    Bucket: "your-bucket-name", // Update with your bucket name
    Key: `invoices/${fileName}`,
    Body: buffer,
    ContentType: "application/pdf",
  };

  try {
    await s3Client.send(new PutObjectCommand(uploadParams));
    return `https://${uploadParams.Bucket}.s3.amazonaws.com/${uploadParams.Key}`;
  } catch (error) {
    console.error("Error uploading PDF to S3: ", error);
    throw error;
  }
}

export async function getPDFFromS3(billNo) {
  const bucketName = "your-bucket-name"; // Update with your bucket name
  const fileName = `invoices/invoice_${billNo}.pdf`;

  const params = {
    Bucket: bucketName,
    Key: fileName,
  };

  try {
    const command = new GetObjectCommand(params);
    const preSignedURL = await getSignedUrl(s3Client, command, { expiresIn: 60 * 5 });
    return preSignedURL;
  } catch (error) {
    console.error("Error fetching PDF from S3:", error);
    return null;
  }
}
```

### 3. Create `generatePDF.js`

Create a file named `generatePDF.js` in the `server/lib` directory and add the following code:

```javascript
import { jsPDF } from "jspdf";
import "jspdf-autotable";
import fs from "fs";
import path from "path";
import { uploadPDFToS3 } from "../services/s3Service.js";

export const generatePDF = async (billData) => {
  const doc = new jsPDF();
  const logoPath = path.join(__dirname, "../generated/xapp.jpeg"); // Correct path to your logo
  let logoImage;

  if (fs.existsSync(logoPath)) {
    const imageBuffer = fs.readFileSync(logoPath);
    logoImage = imageBuffer.toString("base64");
  } else {
    console.error("Logo not found at:", logoPath);
    logoImage = null;
  }

  if (logoImage) {
    doc.addImage(logoImage, "JPEG", 150, 10, 40, 40);
  }

  doc.setFontSize(22);
  doc.text("INVOICE", 10, 20);

  // Add customer and invoice details (update as needed)
  doc.setFontSize(10);
  doc.text("BILL TO", 10, 60);
  doc.text(billData.customer, 10, 65);

  const items = billData.salesDetails.map((item) => [
    item.productName,
    item.quantity,
    `Rs.${item.rate.toFixed(2)}`,
    `Rs.${(item.quantity * item.rate).toFixed(2)}`,
  ]);

  doc.autoTable({
    head: [["DESCRIPTION", "QUANTITY", "UNIT PRICE (Rs.)", "AMOUNT (Rs.)"]],
    body: items,
    startY: 90,
  });

  const pdfBuffer = doc.output('arraybuffer');
  const fileName = `invoice_${billData.billNo}.pdf`;
  const s3Url = await uploadPDFToS3(Buffer.from(pdfBuffer), fileName);
  return s3Url;
};
```

### 4. Create `downloadPDF.js`

Create a file named `downloadPDF.js` in the `server/lib` directory and add the following code:

```javascript
import { getPDFFromS3 } from "../services/s3Service.js";

export async function downloadPDF(req, res) {
  const { billNo } = req.params;

  try {
    const pdfUrl = await getPDFFromS3(billNo);
    if (!pdfUrl) {
      return res.status(404).json({ message: "PDF not found" });
    }
    return res.status(200).json({ url: pdfUrl });
  } catch (error) {
    console.error("Error fetching PDF:", error);
    res.status(500).json({ message: "Internal server error" });
  }
}
```

### 5. Create `PDFRoutes.js`

Create a file named `PDFRoutes.js` in the `server` directory and add the following code:

```javascript
import express from "express";
import { downloadPDF } from './lib/downloadPDF.js';

const router = express.Router();

router.get("/api/pdf/invoice/:billNo", downloadPDF);

export default router;
```

### 6. Create `index.js`

Create a file named `index.js` in the `server` directory and add the following code:

```javascript
import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import PDFRoutes from './PDFRoutes.js';

dotenv.config();

const app = express();
app.use(cors());
app.use(express.json());

app.use(PDFRoutes);

const PORT = process.env.PORT || 8080;

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

## Step 3: Start the Server

To start the server, run the following command in your terminal:

```bash
node index.js
```

You should see the message indicating that the server is running:

```
Server is running on http://localhost:8080
```

## Conclusion

You have successfully set up the server files required for generating PDF invoices and managing uploads to Amazon S3. Your server is now ready to handle requests for PDF generation and retrieval.
