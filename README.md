# Pre-signed PUT URL with Next.js

This project demonstrates how to upload files directly to an S3 bucket from the frontend using a pre-signed PUT URL in a Next.js application.

## Prerequisites

- AWS account with S3 bucket
- AWS SDK
- Next.js application

## Setup

1. **Clone the repository:**

    ```bash
    git clone https://github.com/Ausaf-Vohra/pre-signed-s3
    cd pre-signed-put-url-master
    ```

2. **Install dependencies:**

    ```bash
    npm install
    ```

3. **Configure AWS SDK:**

    Ensure you have your AWS credentials configured. You can use the AWS CLI to configure them:

    ```bash
    aws configure
    ```

## Implementation

### Backend (API Route)

Create an API route in Next.js to generate the pre-signed URL.

```javascript
// pages/api/generate-presigned-url.js
import AWS from 'aws-sdk';

const s3 = new AWS.S3({
  accessKeyId: process.env.AWS_ACCESS_KEY_ID,
  secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  region: process.env.AWS_REGION,
});

export default async (req, res) => {
  const { fileName, fileType } = req.body;

  const s3Params = {
    Bucket: process.env.S3_BUCKET_NAME,
    Key: fileName,
    Expires: 60,
    ContentType: fileType,
  };

  try {
    const signedUrl = await s3.getSignedUrlPromise('putObject', s3Params);
    res.status(200).json({ url: signedUrl });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### Frontend

Create a form to upload files and handle the file upload using the pre-signed URL.

```javascript
// pages/index.js
import { useState } from 'react';

export default function Home() {
  const [file, setFile] = useState(null);

  const handleFileChange = (event) => {
    setFile(event.target.files[0]);
  };

  const handleUpload = async () => {
    if (!file) return;

    const { name, type } = file;

    const res = await fetch('/api/generate-presigned-url', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ fileName: name, fileType: type }),
    });

    const { url } = await res.json();

    await fetch(url, {
      method: 'PUT',
      headers: {
        'Content-Type': type,
      },
      body: file,
    });

    alert('File uploaded successfully!');
  };

  return (
    <div>
      <input type="file" onChange={handleFileChange} />
      <button onClick={handleUpload}>Upload</button>
    </div>
  );
}
```

## Environment Variables

Create a `.env.local` file in the root of your project and add your AWS credentials and S3 bucket name:

```
AWS_ACCESS_KEY_ID=your-access-key-id
AWS_SECRET_ACCESS_KEY=your-secret-access-key
AWS_REGION=your-region
S3_BUCKET_NAME=your-bucket-name
```

## Running the Project

Start the Next.js development server:

```bash
npm run dev
```

Open your browser and navigate to `http://localhost:3000` to test the file upload functionality.

## Conclusion

This project demonstrates how to use pre-signed URLs to securely upload files to an S3 bucket directly from the frontend in a Next.js application.