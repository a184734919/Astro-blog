---
title: Express 文件上传
published: 2023-03-26
description: 'multer 中间件使用的详细介绍和学习笔记'
image: ''
tags: ["Node.js","Express"]
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 文件上传概述

文件上传是 Web 应用的常见功能，包括头像上传、文档分享、图片处理等场景。Express 本身不直接处理文件上传，但通过 `multer` 中间件可以实现强大的文件上传功能。

### 文件上传的重要性

- 用户资料头像上传
- 文档和资料分享
- 图片和媒体文件处理
- 数据备份和导入
- 用户内容生成

### 常见文件上传方案

1. **Multer**: Express 最流行的文件上传中间件
2. **Busboy**: 底层流式处理
3. **Formidable**: 完整的表单和文件处理
4. **前端直传云存储**: 避免服务器负担

## Multer 基础

### 安装和基础配置

```javascript
// 安装 Multer
// npm install multer

const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');

const app = express();

// 基础配置
const upload = multer({ dest: 'uploads/' });

// 单文件上传路由
app.post('/upload', upload.single('file'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }
  
  res.json({
    success: true,
    file: req.file
  });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Multer 文件对象结构

```javascript
{
  fieldname: 'file',              // 表单字段名称
  originalname: 'example.jpg',    // 原始文件名
  encoding: '7bit',               // 文件编码
  mimetype: 'image/jpeg',         // MIME 类型
  size: 102400,                  // 文件大小（字节）
  destination: 'uploads/',       // 保存路径
  filename: 'abc123.jpg',        // 保存的文件名
  path: 'uploads/abc123.jpg',    // 完整文件路径
  buffer: <Buffer 00...>         // 文件内容（内存存储时）
}
```

## 文件存储配置

### 1. 磁盘存储

```javascript
const storage = multer.diskStorage({
  // 设置文件存储目录
  destination: function (req, file, cb) {
    const uploadDir = 'uploads/';
    
    // 确保目录存在
    if (!fs.existsSync(uploadDir)) {
      fs.mkdirSync(uploadDir, { recursive: true });
    }
    
    cb(null, uploadDir);
  },
  
  // 设置文件名
  filename: function (req, file, cb) {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const ext = path.extname(file.originalname);
    const basename = path.basename(file.originalname, ext);
    
    cb(null, basename + '-' + uniqueSuffix + ext);
  }
});

const upload = multer({ storage: storage });
```

### 2. 内存存储

```javascript
const storage = multer.memoryStorage();

const upload = multer({ storage: storage });

app.post('/upload/memory', upload.single('file'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }
  
  // 文件在内存中，可以直接处理
  const fileBuffer = req.file.buffer;
  const fileSize = req.file.size;
  
  // 例如：直接处理图片
  sharp(fileBuffer)
    .resize(300, 300)
    .toBuffer()
    .then(resizedImageBuffer => {
      // 处理后的图片
      res.json({ success: true, size: resizedImageBuffer.length });
    })
    .catch(error => {
      res.status(500).json({ error: 'Image processing failed' });
    });
});
```

### 3. 按日期组织存储

```javascript
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    const date = new Date();
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    
    const uploadDir = path.join('uploads', year, month, day);
    
    if (!fs.existsSync(uploadDir)) {
      fs.mkdirSync(uploadDir, { recursive: true });
    }
    
    cb(null, uploadDir);
  },
  
  filename: function (req, file, cb) {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const ext = path.extname(file.originalname);
    cb(null, uniqueSuffix + ext);
  }
});

const upload = multer({ storage: storage });
```

### 4. 按用户组织存储

```javascript
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    // 假设用户信息在 session 中
    const userId = req.session?.userId || 'anonymous';
    
    const userUploadDir = path.join('uploads', 'users', String(userId));
    
    if (!fs.existsSync(userUploadDir)) {
      fs.mkdirSync(userUploadDir, { recursive: true });
    }
    
    cb(null, userUploadDir);
  },
  
  filename: function (req, file, cb) {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const ext = path.extname(file.originalname);
    cb(null, uniqueSuffix + ext);
  }
});

const upload = multer({ storage: storage });
```

## 文件类型验证

### 1. MIME 类型过滤

```javascript
// 允许的 MIME 类型
const allowedMimeTypes = [
  'image/jpeg',
  'image/png',
  'image/gif',
  'image/webp',
  'application/pdf',
  'text/plain',
  'application/msword',
  'application/vnd.openxmlformats-officedocument.wordprocessingml.document'
];

const upload = multer({
  storage: storage,
  fileFilter: function (req, file, cb) {
    if (allowedMimeTypes.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error('Invalid file type'), false);
    }
  }
});
```

### 2. 文件扩展名过滤

```javascript
const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif', '.webp', '.pdf', '.doc', '.docx', '.txt'];

const upload = multer({
  storage: storage,
  fileFilter: function (req, file, cb) {
    const ext = path.extname(file.originalname).toLowerCase();
    
    if (allowedExtensions.includes(ext)) {
      cb(null, true);
    } else {
      cb(new Error('Invalid file extension'), false);
    }
  }
});
```

### 3. 综合文件验证

```javascript
const fileFilter = (req, file, cb) => {
  const ext = path.extname(file.originalname).toLowerCase();
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif'];
  const allowedMimeTypes = ['image/jpeg', 'image/png', 'image/gif'];
  
  // 检查扩展名
  if (!allowedExtensions.includes(ext)) {
    return cb(new Error('Invalid file extension'), false);
  }
  
  // 检查 MIME 类型
  if (!allowedMimeTypes.includes(file.mimetype)) {
    return cb(new Error('Invalid file type'), false);
  }
  
  cb(null, true);
};

const upload = multer({
  storage: storage,
  fileFilter: fileFilter
});
```

### 4. 动态文件验证

```javascript
const dynamicFileFilter = (fileTypes) => {
  return (req, file, cb) => {
    if (fileTypes.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error(`Invalid file type. Allowed types: ${fileTypes.join(', ')}`), false);
    }
  };
};

// 使用动态过滤器
const imageUpload = multer({
  storage: storage,
  fileFilter: dynamicFileFilter(['image/jpeg', 'image/png', 'image/gif'])
});

const documentUpload = multer({
  storage: storage,
  fileFilter: dynamicFileFilter(['application/pdf', 'application/msword'])
});
```

## 文件大小限制

### 基础大小限制

```javascript
const upload = multer({
  storage: storage,
  limits: {
    fileSize: 5 * 1024 * 1024 // 5MB
  }
});
```

### 不同类型文件的大小限制

```javascript
const sizeLimits = {
  image: 2 * 1024 * 1024,      // 2MB
  document: 10 * 1024 * 1024,  // 10MB
  video: 50 * 1024 * 1024      // 50MB
};

const createUploadMiddleware = (type) => {
  return multer({
    storage: storage,
    limits: {
      fileSize: sizeLimits[type] || 5 * 1024 * 1024
    },
    fileFilter: fileFilter
  });
};

const imageUpload = createUploadMiddleware('image');
const documentUpload = createUploadMiddleware('document');
const videoUpload = createUploadMiddleware('video');

app.post('/upload/image', imageUpload.single('image'), (req, res) => {
  res.json({ success: true, file: req.file });
});

app.post('/upload/document', documentUpload.single('document'), (req, res) => {
  res.json({ success: true, file: req.file });
});
```

### 基于用户角色的大小限制

```javascript
const createRoleBasedUpload = (role) => {
  const limits = {
    admin: 100 * 1024 * 1024,    // 100MB
    premium: 50 * 1024 * 1024,   // 50MB
    user: 10 * 1024 * 1024       // 10MB
  };
  
  return multer({
    storage: storage,
    limits: {
      fileSize: limits[role] || limits.user
    }
  });
};

const upload = createRoleBasedUpload('user'); // 默认用户限制
```

## 多文件上传

### 基础多文件上传

```javascript
// 上传多个文件（相同字段）
app.post('/upload/multiple', upload.array('files', 10), (req, res) => {
  if (!req.files || req.files.length === 0) {
    return res.status(400).json({ error: 'No files uploaded' });
  }
  
  res.json({
    success: true,
    files: req.files,
    count: req.files.length
  });
});
```

### 不同字段多文件上传

```javascript
// 不同字段的上传
const multiFieldUpload = upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'documents', maxCount: 5 },
  { name: 'gallery', maxCount: 10 }
]);

app.post('/upload/multi-field', multiFieldUpload, (req, res) => {
  const { avatar, documents, gallery } = req.files;
  
  res.json({
    success: true,
    files: {
      avatar: avatar?.[0],
      documents: documents || [],
      gallery: gallery || []
    }
  });
});
```

### 动态多文件上传

```javascript
// 上传任意数量的文件
app.post('/upload/any', upload.any(), (req, res) => {
  if (!req.files || req.files.length === 0) {
    return res.status(400).json({ error: 'No files uploaded' });
  }
  
  res.json({
    success: true,
    files: req.files
  });
});
```

## 图片处理

### 基础图片处理

```javascript
const sharp = require('sharp');

const imageStorage = multer.diskStorage({
  destination: function (req, file, cb) {
    const uploadDir = 'uploads/images/';
    if (!fs.existsSync(uploadDir)) {
      fs.mkdirSync(uploadDir, { recursive: true });
    }
    cb(null, uploadDir);
  },
  filename: function (req, file, cb) {
    const uniqueSuffix = Date.now();
    cb(null, uniqueSuffix + path.extname(file.originalname));
  }
});

const imageUpload = multer({
  storage: imageStorage,
  fileFilter: (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) {
      cb(null, true);
    } else {
      cb(new Error('Only image files are allowed'), false);
    }
  }
});

app.post('/upload/image', imageUpload.single('image'), async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No image uploaded' });
    }
    
    const originalPath = req.file.path;
    const thumbnailPath = `uploads/images/thumbnail-${req.file.filename}`;
    const compressedPath = `uploads/images/compressed-${req.file.filename}`;
    
    // 生成缩略图
    await sharp(originalPath)
      .resize(200, 200, { fit: 'cover' })
      .toFile(thumbnailPath);
    
    // 压缩图片
    await sharp(originalPath)
      .jpeg({ quality: 80 })
      .toFile(compressedPath);
    
    // 删除原始文件（可选）
    // fs.unlinkSync(originalPath);
    
    res.json({
      success: true,
      images: {
        original: originalPath,
        thumbnail: thumbnailPath,
        compressed: compressedPath
      }
    });
  } catch (error) {
    console.error('Image processing error:', error);
    res.status(500).json({ error: 'Image processing failed' });
  }
});
```

### 图片格式转换

```javascript
app.post('/upload/convert', imageUpload.single('image'), async (req, res) => {
  try {
    const { format = 'webp' } = req.body;
    const supportedFormats = ['jpeg', 'png', 'webp', 'gif'];
    
    if (!supportedFormats.includes(format)) {
      return res.status(400).json({ error: 'Unsupported format' });
    }
    
    const outputFileName = req.file.filename.replace(/\.[^/.]+$/, `.${format}`);
    const outputPath = path.join('uploads/images', outputFileName);
    
    await sharp(req.file.path)
      .toFormat(format)
      .toFile(outputPath);
    
    res.json({
      success: true,
      convertedImage: outputPath,
      format: format
    });
  } catch (error) {
    console.error('Conversion error:', error);
    res.status(500).json({ error: 'Image conversion failed' });
  }
});
```

### 图片水印

```javascript
app.post('/upload/watermark', imageUpload.single('image'), async (req, res) => {
  try {
    const { text = 'Sample Watermark' } = req.body;
    const outputPath = path.join('uploads/images', 'watermarked-' + req.file.filename);
    
    await sharp(req.file.path)
      .resize(1200)
      .composite([{
        input: {
          text: {
            text: text,
            font: 'Arial',
            size: 40,
            color: 'rgba(255, 255, 255, 0.5)'
          }
        },
        gravity: 'southeast'
      }])
      .toFile(outputPath);
    
    res.json({
      success: true,
      watermarkedImage: outputPath
    });
  } catch (error) {
    console.error('Watermark error:', error);
    res.status(500).json({ error: 'Watermark processing failed' });
  }
});
```

## 错误处理

### Multer 错误处理

```javascript
// 错误处理中间件
app.use((error, req, res, next) => {
  if (error instanceof multer.MulterError) {
    if (error.code === 'LIMIT_FILE_SIZE') {
      return res.status(400).json({
        error: 'File size exceeds limit',
        maxSize: '5MB'
      });
    }
    
    if (error.code === 'LIMIT_FILE_COUNT') {
      return res.status(400).json({
        error: 'Too many files',
        maxFiles: 10
      });
    }
    
    if (error.code === 'LIMIT_UNEXPECTED_FILE') {
      return res.status(400).json({
        error: 'Unexpected file field'
      });
    }
    
    return res.status(400).json({
      error: 'File upload error',
      details: error.message
    });
  }
  
  if (error.message === 'Invalid file type') {
    return res.status(400).json({
      error: 'Invalid file type',
      allowedTypes: ['image/jpeg', 'image/png', 'image/gif']
    });
  }
  
  next(error);
});
```

### 完整的错误处理示例

```javascript
const upload = multer({
  storage: storage,
  limits: { fileSize: 5 * 1024 * 1024 },
  fileFilter: fileFilter
});

app.post('/upload', upload.single('file'), async (req, res, next) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }
    
    // 处理文件
    await processFile(req.file);
    
    res.json({
      success: true,
      file: req.file
    });
  } catch (error) {
    // 清理上传的文件
    if (req.file && req.file.path) {
      fs.unlink(req.file.path, (err) => {
        if (err) console.error('File cleanup error:', err);
      });
    }
    next(error);
  }
});

app.use((error, req, res, next) => {
  console.error('Upload error:', error);
  res.status(500).json({
    error: 'Upload failed',
    message: process.env.NODE_ENV === 'development' ? error.message : undefined
  });
});
```

## 安全性增强

### 1. 文件名安全处理

```javascript
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    const uploadDir = 'uploads/';
    if (!fs.existsSync(uploadDir)) {
      fs.mkdirSync(uploadDir, { recursive: true });
    }
    cb(null, uploadDir);
  },
  
  filename: function (req, file, cb) {
    // 移除特殊字符，只保留字母、数字、点、下划线和连字符
    const safeName = file.originalname
      .replace(/[^a-zA-Z0-9._-]/g, '')
      .toLowerCase();
    
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const ext = path.extname(safeName);
    const basename = path.basename(safeName, ext);
    
    cb(null, `${basename}-${uniqueSuffix}${ext}`);
  }
});
```

### 2. 文件内容验证

```javascript
const fileType = require('file-type');
const { readChunk } = require('file-type/core');

const verifyFileContent = async (filePath, allowedMimeTypes) => {
  const buffer = readChunk.sync(filePath, 0, 4100);
  const type = await fileType.fromBuffer(buffer);
  
  if (!type || !allowedMimeTypes.includes(type.mime)) {
    throw new Error('Invalid file content');
  }
  
  return type.mime;
};

app.post('/upload/verified', upload.single('file'), async (req, res, next) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }
    
    // 验证文件内容
    const actualMimeType = await verifyFileContent(
      req.file.path,
      ['image/jpeg', 'image/png', 'image/gif']
    );
    
    console.log('Verified MIME type:', actualMimeType);
    
    res.json({
      success: true,
      file: req.file,
      verifiedMimeType: actualMimeType
    });
  } catch (error) {
    // 清理文件
    if (req.file && req.file.path) {
      fs.unlink(req.file.path, (err) => {
        if (err) console.error('File cleanup error:', err);
      });
    }
    next(error);
  }
});
```

### 3. 病毒扫描（集成 ClamAV）

```javascript
const NodeClam = require('clamav.js');

const clamav = new NodeClam().init({
  clamdscan: {
    host: 'localhost',
    port: 3310,
    timeout: 60000,
    local_fallback: true
  },
  preference: 'clamdscan'
});

app.post('/upload/secure', upload.single('file'), async (req, res, next) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }
    
    // 病毒扫描
    const result = await clamav.scanFile(req.file.path);
    
    if (result.isInfected) {
      // 删除感染的文件
      fs.unlink(req.file.path, (err) => {
        if (err) console.error('File deletion error:', err);
      });
      
      return res.status(400).json({
        error: 'File contains virus',
        viruses: result.viruses
      });
    }
    
    res.json({
      success: true,
      file: req.file,
      scanResult: 'clean'
    });
  } catch (error) {
    next(error);
  }
});
```

## 文件管理接口

### 文件列表

```javascript
app.get('/files', async (req, res) => {
  try {
    const { page = 1, limit = 10, type } = req.query;
    
    let files = [];
    const uploadDir = 'uploads/';
    
    if (fs.existsSync(uploadDir)) {
      const allFiles = fs.readdirSync(uploadDir);
      
      // 过滤文件类型
      if (type) {
        files = allFiles.filter(file => file.endsWith(type));
      } else {
        files = allFiles;
      }
      
      // 分页
      const startIndex = (page - 1) * limit;
      const endIndex = startIndex + parseInt(limit);
      
      files = files.slice(startIndex, endIndex).map(filename => {
        const filePath = path.join(uploadDir, filename);
        const stats = fs.statSync(filePath);
        
        return {
          filename,
          size: stats.size,
          uploadDate: stats.mtime,
          url: `/uploads/${filename}`
        };
      });
    }
    
    res.json({
      success: true,
      files,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total: files.length
      }
    });
  } catch (error) {
    console.error('File list error:', error);
    res.status(500).json({ error: 'Failed to list files' });
  }
});
```

### 文件删除

```javascript
app.delete('/files/:filename', (req, res) => {
  try {
    const { filename } = req.params;
    const filePath = path.join('uploads/', filename);
    
    if (!fs.existsSync(filePath)) {
      return res.status(404).json({ error: 'File not found' });
    }
    
    fs.unlinkSync(filePath);
    
    res.json({
      success: true,
      message: 'File deleted successfully'
    });
  } catch (error) {
    console.error('File deletion error:', error);
    res.status(500).json({ error: 'Failed to delete file' });
  }
});
```

### 批量文件删除

```javascript
app.delete('/files/batch', (req, res) => {
  try {
    const { filenames } = req.body;
    
    if (!Array.isArray(filenames)) {
      return res.status(400).json({ error: 'Invalid filenames' });
    }
    
    const deletedFiles = [];
    const failedFiles = [];
    
    filenames.forEach(filename => {
      try {
        const filePath = path.join('uploads/', filename);
        if (fs.existsSync(filePath)) {
          fs.unlinkSync(filePath);
          deletedFiles.push(filename);
        } else {
          failedFiles.push({ filename, error: 'File not found' });
        }
      } catch (error) {
        failedFiles.push({ filename, error: error.message });
      }
    });
    
    res.json({
      success: true,
      deletedFiles,
      failedFiles
    });
  } catch (error) {
    console.error('Batch deletion error:', error);
    res.status(500).json({ error: 'Batch deletion failed' });
  }
});
```

## 前端上传示例

### HTML 表单上传

```html
<!DOCTYPE html>
<html>
<head>
  <title>File Upload</title>
</head>
<body>
  <h2>File Upload</h2>
  
  <form id="uploadForm" enctype="multipart/form-data">
    <div>
      <label for="file">Choose file:</label>
      <input type="file" id="file" name="file" required>
      <span id="fileName"></span>
    </div>
    
    <div>
      <progress id="uploadProgress" value="0" max="100"></progress>
      <span id="progressText">0%</span>
    </div>
    
    <button type="submit">Upload</button>
  </form>
  
  <div id="result"></div>
  
  <script>
    const uploadForm = document.getElementById('uploadForm');
    const fileInput = document.getElementById('file');
    const fileName = document.getElementById('fileName');
    const uploadProgress = document.getElementById('uploadProgress');
    const progressText = document.getElementById('progressText');
    const result = document.getElementById('result');
    
    fileInput.addEventListener('change', () => {
      fileName.textContent = fileInput.files[0]?.name || '';
    });
    
    uploadForm.addEventListener('submit', async (e) => {
      e.preventDefault();
      
      const formData = new FormData();
      formData.append('file', fileInput.files[0]);
      
      try {
        const response = await fetch('/upload', {
          method: 'POST',
          body: formData,
          
          onUploadProgress: (progressEvent) => {
            const percentCompleted = Math.round(
              (progressEvent.loaded * 100) / progressEvent.total
            );
            uploadProgress.value = percentCompleted;
            progressText.textContent = `${percentCompleted}%`;
          }
        });
        
        const data = await response.json();
        
        if (response.ok) {
          result.innerHTML = `
            <div style="color: green;">
              Upload successful!
              <p>File: ${data.file.originalname}</p>
              <p>Size: ${data.file.size} bytes</p>
            </div>
          `;
        } else {
          result.innerHTML = `
            <div style="color: red;">
              Upload failed: ${data.error}
            </div>
          `;
        }
      } catch (error) {
        result.innerHTML = `
          <div style="color: red;">
            Upload error: ${error.message}
          </div>
        `;
      }
    });
  </script>
</body>
</html>
```

### 拖拽上传

```html
<!DOCTYPE html>
<html>
<head>
  <title>Drag and Drop Upload</title>
  <style>
    #dropZone {
      width: 300px;
      height: 200px;
      border: 2px dashed #ccc;
      display: flex;
      align-items: center;
      justify-content: center;
      text-align: center;
    }
    
    #dropZone.dragover {
      border-color: #4CAF50;
      background-color: #e8f5e8;
    }
  </style>
</head>
<body>
  <h2>Drag and Drop Upload</h2>
  
  <div id="dropZone">
    <p>Drop files here to upload</p>
  </div>
  
  <div id="uploadStatus"></div>
  
  <script>
    const dropZone = document.getElementById('dropZone');
    const uploadStatus = document.getElementById('uploadStatus');
    
    // 阻止默认行为
    ['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => {
      dropZone.addEventListener(eventName, preventDefaults, false);
    });
    
    function preventDefaults(e) {
      e.preventDefault();
      e.stopPropagation();
    }
    
    // 添加样式
    ['dragenter', 'dragover'].forEach(eventName => {
      dropZone.addEventListener(eventName, () => {
        dropZone.classList.add('dragover');
      }, false);
    });
    
    ['dragleave', 'drop'].forEach(eventName => {
      dropZone.addEventListener(eventName, () => {
        dropZone.classList.remove('dragover');
      }, false);
    });
    
    // 处理文件拖放
    dropZone.addEventListener('drop', handleDrop, false);
    
    function handleDrop(e) {
      const files = e.dataTransfer.files;
      
      if (files.length > 0) {
        uploadFile(files[0]);
      }
    }
    
    async function uploadFile(file) {
      const formData = new FormData();
      formData.append('file', file);
      
      uploadStatus.innerHTML = 'Uploading...';
      
      try {
        const response = await fetch('/upload', {
          method: 'POST',
          body: formData
        });
        
        const data = await response.json();
        
        if (response.ok) {
          uploadStatus.innerHTML = `
            <div style="color: green;">
              Upload successful!
              <p>File: ${data.file.originalname}</p>
              <p>Size: ${data.file.size} bytes</p>
            </div>
          `;
        } else {
          uploadStatus.innerHTML = `
            <div style="color: red;">
              Upload failed: ${data.error}
            </div>
          `;
        }
      } catch (error) {
        uploadStatus.innerHTML = `
          <div style="color: red;">
            Upload error: ${error.message}
          </div>
        `;
      }
    }
  </script>
</body>
</html>
```

## 最佳实践

### 1. 文件清理

```javascript
// 定期清理旧文件
const cleanupOldFiles = () => {
  const uploadDir = 'uploads/';
  const maxAge = 7 * 24 * 60 * 60 * 1000; // 7 天
  const now = Date.now();
  
  try {
    const files = fs.readdirSync(uploadDir);
    
    files.forEach(file => {
      const filePath = path.join(uploadDir, file);
      const stats = fs.statSync(filePath);
      const fileAge = now - stats.mtimeMs;
      
      if (fileAge > maxAge) {
        fs.unlinkSync(filePath);
        console.log('Deleted old file:', file);
      }
    });
  } catch (error) {
    console.error('Cleanup error:', error);
  }
};

// 每天执行一次清理
setInterval(cleanupOldFiles, 24 * 60 * 60 * 1000);
```

### 2. 文件监控

```javascript
const chokidar = require('chokidar');

// 监控上传目录
const watcher = chokidar.watch('uploads/', {
  ignored: /(^|[\/\\])\../, // 忽略点文件
  persistent: true
});

watcher
  .on('add', (path, stats) => {
    console.log(`File added: ${path}, size: ${stats?.size} bytes`);
  })
  .on('change', (path, stats) => {
    console.log(`File changed: ${path}, size: ${stats?.size} bytes`);
  })
  .on('unlink', (path) => {
    console.log(`File deleted: ${path}`);
  });
```

### 3. 文件统计

```javascript
app.get('/files/stats', async (req, res) => {
  try {
    const uploadDir = 'uploads/';
    let totalSize = 0;
    let fileCount = 0;
    const fileTypeStats = {};
    
    if (fs.existsSync(uploadDir)) {
      const files = fs.readdirSync(uploadDir);
      
      files.forEach(file => {
        const filePath = path.join(uploadDir, file);
        const stats = fs.statSync(filePath);
        const ext = path.extname(file).toLowerCase();
        
        totalSize += stats.size;
        fileCount++;
        
        fileTypeStats[ext] = (fileTypeStats[ext] || 0) + 1;
      });
    }
    
    res.json({
      success: true,
      stats: {
        totalSize,
        fileCount,
        fileTypeStats,
        averageFileSize: fileCount > 0 ? totalSize / fileCount : 0
      }
    });
  } catch (error) {
    console.error('Stats error:', error);
    res.status(500).json({ error: 'Failed to get file stats' });
  }
});
```

## 常见问题和解决方案

### 问题 1：文件上传大小限制

```javascript
// 问题：文件上传失败，提示 "Request entity too large"

// 解决方案：增加 Express 主体限制
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ limit: '10mb', extended: true }));

// 配置 Multer 限制
const upload = multer({
  limits: {
    fileSize: 50 * 1024 * 1024 // 50MB
  }
});
```

### 问题 2：临时文件未清理

```javascript
// 问题：上传失败后临时文件未被删除

// 解决方案：在错误处理中清理文件
app.post('/upload', upload.single('file'), async (req, res, next) => {
  try {
    // 处理文件
    await processFile(req.file);
    res.json({ success: true });
  } catch (error) {
    // 清理文件
    if (req.file && req.file.path) {
      fs.unlink(req.file.path, (err) => {
        if (err) console.error('Cleanup error:', err);
      });
    }
    next(error);
  }
});
```

### 问题 3：文件名冲突

```javascript
// 问题：相同文件名的文件相互覆盖

// 解决方案：使用唯一文件名
const storage = multer.diskStorage({
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const ext = path.extname(file.originalname);
    cb(null, uniqueSuffix + ext);
  }
});
```

## 总结

Express 文件上传通过 Multer 中间件提供了强大而灵活的功能。通过本篇详细指南，你已经了解了：

- Multer 的基础概念和配置
- 不同的文件存储方案（磁盘、内存）
- 文件类型和大小验证
- 单文件和多文件上传
- 图片处理和格式转换
- 完整的错误处理机制
- 安全性增强措施
- 文件管理接口
- 前端上传实现
- 文件清理和监控
- 最佳实践和常见问题解决

在实际项目中，建议：

1. 根据应用需求选择合适的存储方案
2. 实施严格的文件验证和安全措施
3. 定期清理临时文件和过期文件
4. 监控文件上传活动
5. 实现文件统计和管理功能
6. 考虑集成云存储服务

掌握 Express 文件上传功能能够为应用提供丰富的文件处理能力，满足各种业务需求。