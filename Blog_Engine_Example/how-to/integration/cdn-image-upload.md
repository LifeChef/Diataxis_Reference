# Integrate CDN for Image Uploads

> **Document Type**: How-To Guide (SAMPLE)  
> **Audience**: Backend Developers, Frontend Developers  
> **Difficulty**: Advanced  
> **Last Updated**: November 5, 2024

---

## Problem

You need to implement image upload functionality using AWS S3 and CloudFront CDN for optimal performance and scalability.

---

## Prerequisites

- AWS account with S3 and CloudFront access
- AWS credentials configured
- Node.js backend service

---

## Solution

### Step 1: Install AWS SDK

```bash
npm install aws-sdk multer multer-s3
```

### Step 2: Configure S3 Bucket

Create S3 bucket with proper CORS configuration:

```json
{
  "CORSRules": [{
    "AllowedOrigins": ["https://blogengine.com"],
    "AllowedMethods": ["GET", "PUT", "POST"],
    "AllowedHeaders": ["*"],
    "MaxAgeSeconds": 3000
  }]
}
```

### Step 3: Set Up CloudFront Distribution

Point CloudFront to your S3 bucket for cached delivery.

### Step 4: Implement Upload Service

Create `src/services/upload.service.js`:

```javascript
const AWS = require('aws-sdk');
const multer = require('multer');
const multerS3 = require('multer-s3');
const path = require('path');

const s3 = new AWS.S3({
  accessKeyId: process.env.AWS_ACCESS_KEY,
  secretAccessKey: process.env.AWS_SECRET_KEY,
  region: process.env.AWS_REGION
});

const upload = multer({
  storage: multerS3({
    s3: s3,
    bucket: process.env.S3_BUCKET,
    acl: 'public-read',
    metadata: (req, file, cb) => {
      cb(null, { fieldName: file.fieldname });
    },
    key: (req, file, cb) => {
      const fileName = `${Date.now()}-${file.originalname}`;
      cb(null, `uploads/${fileName}`);
    }
  }),
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB limit
  fileFilter: (req, file, cb) => {
    const allowedTypes = /jpeg|jpg|png|gif/;
    const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
    const mimetype = allowedTypes.test(file.mimetype);
    
    if (extname && mimetype) {
      cb(null, true);
    } else {
      cb(new Error('Only image files allowed'));
    }
  }
});

module.exports = upload;
```

### Step 5: Create Upload Endpoint

```javascript
const express = require('express');
const upload = require('./services/upload.service');

router.post('/api/upload', upload.single('image'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }
  
  const cdnUrl = `${process.env.CDN_URL}/${req.file.key}`;
  
  res.json({
    url: cdnUrl,
    size: req.file.size,
    mimetype: req.file.mimetype
  });
});
```

### Step 6: Frontend Implementation

```javascript
const handleImageUpload = async (file) => {
  const formData = new FormData();
  formData.append('image', file);
  
  const response = await fetch('/api/upload', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`
    },
    body: formData
  });
  
  const { url } = await response.json();
  return url; // CloudFront CDN URL
};
```

---

## Verification

1. Upload test image
2. Verify file appears in S3 bucket
3. Access image via CloudFront URL
4. Check image loads quickly (< 200ms)

---

## Troubleshooting

**Upload fails with CORS error:**
- Verify S3 CORS configuration
- Check allowed origins include your domain

**Image not accessible:**
- Verify S3 bucket permissions
- Check CloudFront distribution status

**Slow upload speed:**
- Consider direct S3 upload from frontend
- Implement multipart upload for large files

---

## Related

- [Post Service Design](../../docs/architecture/post-service.md)
- [AWS S3 Documentation](https://docs.aws.amazon.com/s3/)
- [Image Optimization Guide](image-optimization.md)
