---
title: Node.js 核心模块 crypto
published: 2023-01-30
description: '加密和解密操作的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Crypto 是 Node.js 的核心模块之一，提供了 OpenSSL 的封装，用于实现各种加密、解密、哈希和签名算法。该模块在处理敏感数据、实现安全通信和数据完整性校验等场景中发挥着重要作用。

## 核心概念

### 加密算法类型

1. **哈希算法** - 单向加密，用于数据完整性校验
   - MD5、SHA-1、SHA-256、SHA-512 等

2. **对称加密** - 使用相同密钥进行加密和解密
   - AES、DES、Triple DES 等

3. **非对称加密** - 使用公钥和私钥
   - RSA、DSA、ECDSA 等

4. **HMAC** - 基于哈希的消息认证码

### 密钥管理

- **密钥生成**：使用安全随机数生成器生成密钥
- **密钥派生**：从密码派生加密密钥
- **密钥交换**：在不安全的通道上安全地交换密钥

### 编码格式

- **Hex**：十六进制编码
- **Base64**：Base64 编码
- **Binary**：二进制格式

## 基本用法

### 哈希函数

```javascript
const crypto = require('crypto');

// MD5 哈希
const md5Hash = crypto.createHash('md5').update('Hello World').digest('hex');
console.log('MD5:', md5Hash);

// SHA-256 哈希
const sha256Hash = crypto.createHash('sha256').update('Hello World').digest('hex');
console.log('SHA-256:', sha256Hash);

// SHA-512 哈希
const sha512Hash = crypto.createHash('sha512').update('Hello World').digest('hex');
console.log('SHA-512:', sha512Hash);

// 流式哈希计算（适合大文件）
const hash = crypto.createHash('sha256');
hash.update('Hello ');
hash.update('World');
const streamHash = hash.digest('hex');
console.log('流式哈希:', streamHash);

// 不同编码格式
const hash = crypto.createHash('sha256').update('Hello World');
console.log('Hex:', hash.digest('hex'));
console.log('Base64:', hash.digest('base64'));
console.log('Binary:', hash.digest('binary'));
```

### 对称加密（AES）

```javascript
const crypto = require('crypto');

// 生成密钥和初始化向量
const algorithm = 'aes-256-cbc';
const key = crypto.randomBytes(32); // 256 位密钥
const iv = crypto.randomBytes(16);  // 128 位初始化向量

// 加密函数
function encrypt(text, key, iv) {
  const cipher = crypto.createCipheriv(algorithm, key, iv);
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  return encrypted;
}

// 解密函数
function decrypt(encrypted, key, iv) {
  const decipher = crypto.createDecipheriv(algorithm, key, iv);
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
}

// 使用示例
const plaintext = '这是要加密的秘密信息';
const encrypted = encrypt(plaintext, key, iv);
const decrypted = decrypt(encrypted, key, iv);

console.log('原文:', plaintext);
console.log('密文:', encrypted);
console.log('解密:', decrypted);

// 使用认证加密模式（GCM）
const keyGCM = crypto.randomBytes(32);
const ivGCM = crypto.randomBytes(12);
const tag = Buffer.alloc(16);

const cipherGCM = crypto.createCipheriv('aes-256-gcm', keyGCM, ivGCM);
let encryptedGCM = cipherGCM.update('敏感数据', 'utf8', 'hex');
encryptedGCM += cipherGCM.final('hex');
const authTag = cipherGCM.getAuthTag();

console.log('GCM 加密:', encryptedGCM);
console.log('认证标签:', authTag.toString('hex'));
```

### 非对称加密（RSA）

```javascript
const crypto = require('crypto');

// 生成 RSA 密钥对
const { publicKey, privateKey } = crypto.generateKeyPairSync('rsa', {
  modulusLength: 2048,
  publicKeyEncoding: {
    type: 'spki',
    format: 'pem'
  },
  privateKeyEncoding: {
    type: 'pkcs8',
    format: 'pem'
  }
});

console.log('公钥:', publicKey);
console.log('私钥:', privateKey);

// 使用公钥加密
function encryptWithPublicKey(data, publicKey) {
  const buffer = Buffer.from(data, 'utf8');
  const encrypted = crypto.publicEncrypt(
    {
      key: publicKey,
      padding: crypto.constants.RSA_PKCS1_OAEP_PADDING,
      oaepHash: 'sha256'
    },
    buffer
  );
  return encrypted.toString('base64');
}

// 使用私钥解密
function decryptWithPrivateKey(encryptedData, privateKey) {
  const buffer = Buffer.from(encryptedData, 'base64');
  const decrypted = crypto.privateDecrypt(
    {
      key: privateKey,
      padding: crypto.constants.RSA_PKCS1_OAEP_PADDING,
      oaepHash: 'sha256'
    },
    buffer
  );
  return decrypted.toString('utf8');
}

// 使用示例
const message = '使用 RSA 加密的数据';
const encrypted = encryptWithPublicKey(message, publicKey);
const decrypted = decryptWithPrivateKey(encrypted, privateKey);

console.log('原文:', message);
console.log('加密:', encrypted);
console.log('解密:', decrypted);

// 数字签名
function sign(data, privateKey) {
  const sign = crypto.createSign('SHA256');
  sign.update(data);
  sign.end();
  return sign.sign(privateKey, 'hex');
}

function verify(data, signature, publicKey) {
  const verify = crypto.createVerify('SHA256');
  verify.update(data);
  verify.end();
  return verify.verify(publicKey, signature, 'hex');
}

const data = '待签名的数据';
const signature = sign(data, privateKey);
const isValid = verify(data, signature, publicKey);

console.log('签名:', signature);
console.log('验证结果:', isValid);
```

### HMAC

```javascript
const crypto = require('crypto');

const secret = 'my-secret-key';
const message = '要认证的消息';

// 创建 HMAC
const hmac = crypto.createHmac('sha256', secret);
hmac.update(message);
const hmacDigest = hmac.digest('hex');

console.log('HMAC:', hmacDigest);

// 比较函数（防止时序攻击）
const receivedHmac = 'abc123'; // 从外部接收的 HMAC
const isValidHmac = crypto.timingSafeEqual(
  Buffer.from(hmacDigest, 'hex'),
  Buffer.from(receivedHmac, 'hex')
);

console.log('HMAC 验证:', isValidHmac);
```

### 密码哈希（bcrypt）

```javascript
const crypto = require('crypto');

// 使用 scrypt 进行密码哈希（推荐的现代方法）
function hashPassword(password, salt) {
  return new Promise((resolve, reject) => {
    crypto.scrypt(password, salt, 64, (err, derivedKey) => {
      if (err) reject(err);
      else resolve(derivedKey.toString('hex'));
    });
  });
}

// 使用示例
async function passwordExample() {
  const password = 'user123';
  const salt = crypto.randomBytes(16).toString('hex');

  const hashedPassword = await hashPassword(password, salt);
  console.log('密码哈希:', hashedPassword);

  // 验证密码
  const verifyHash = await hashPassword(password, salt);
  const isValid = hashedPassword === verifyHash;
  console.log('密码验证:', isValid);
}

passwordExample();

// 生成随机盐
const salt = crypto.randomBytes(16);
console.log('盐值:', salt.toString('hex'));
```

### 随机数生成

```javascript
const crypto = require('crypto');

// 生成随机字节
const randomBytes = crypto.randomBytes(16);
console.log('随机字节:', randomBytes.toString('hex'));

// 生成随机整数
const randomInt = crypto.randomInt(0, 100);
console.log('随机整数:', randomInt);

// 生成随机字符串
function generateRandomString(length) {
  const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  let result = '';
  for (let i = 0; i < length; i++) {
    result += characters.charAt(Math.floor(Math.random() * characters.length));
  }
  return result;
}

console.log('随机字符串:', generateRandomString(16));
```

## 实际应用

### 密码存储

```javascript
const crypto = require('crypto');

// 安全的密码哈希函数
async function hashPassword(password) {
  const salt = crypto.randomBytes(16);
  const iterations = 100000;
  const keylen = 64;
  const digest = 'sha256';

  return new Promise((resolve, reject) => {
    crypto.pbkdf2(password, salt, iterations, keylen, digest, (err, derivedKey) => {
      if (err) reject(err);
      else {
        const hash = `${iterations}$${salt.toString('hex')}$${derivedKey.toString('hex')}`;
        resolve(hash);
      }
    });
  });
}

// 验证密码
async function verifyPassword(password, hash) {
  const [iterations, salt, key] = hash.split('$');

  return new Promise((resolve, reject) => {
    crypto.pbkdf2(password, Buffer.from(salt, 'hex'), parseInt(iterations), 64, 'sha256', (err, derivedKey) => {
      if (err) reject(err);
      else resolve(key === derivedKey.toString('hex'));
    });
  });
}

// 使用示例
async function passwordStorageExample() {
  const password = 'SecurePassword123!';
  const hashedPassword = await hashPassword(password);
  console.log('存储的哈希:', hashedPassword);

  const isValid = await verifyPassword('SecurePassword123!', hashedPassword);
  const isInvalid = await verifyPassword('WrongPassword', hashedPassword);

  console.log('正确密码验证:', isValid);
  console.log('错误密码验证:', isInvalid);
}

passwordStorageExample();
```

### 数据签名和验证

```javascript
const crypto = require('crypto');

// 生成密钥对
const { publicKey, privateKey } = crypto.generateKeyPairSync('rsa', {
  modulusLength: 2048
});

// 签名函数
function signObject(obj, privateKey) {
  const data = JSON.stringify(obj);
  const sign = crypto.createSign('SHA256');
  sign.update(data);
  return sign.sign(privateKey, 'hex');
}

// 验证函数
function verifyObject(obj, signature, publicKey) {
  const data = JSON.stringify(obj);
  const verify = crypto.createVerify('SHA256');
  verify.update(data);
  return verify.verify(publicKey, signature, 'hex');
}

// 创建签名 API 请求
async function createSignedRequest(data, privateKey) {
  const timestamp = Date.now();
  const payload = { ...data, timestamp };
  const signature = signObject(payload, privateKey);

  return {
    payload,
    signature
  };
}

// 验证 API 请求
async function verifySignedRequest(request, publicKey) {
  const { payload, signature } = request;
  const isValid = verifyObject(payload, signature, publicKey);

  // 检查时间戳，防止重放攻击
  const timestamp = payload.timestamp;
  const currentTime = Date.now();
  const timeDifference = currentTime - timestamp;

  if (timeDifference > 300000) { // 5 分钟过期
    return { valid: false, reason: 'Request expired' };
  }

  return { valid: isValid };
}

// 使用示例
async function apiExample() {
  const apiData = { userId: 123, action: 'transfer', amount: 1000 };

  const signedRequest = await createSignedRequest(apiData, privateKey);
  console.log('签名请求:', signedRequest);

  const verification = await verifySignedRequest(signedRequest, publicKey);
  console.log('验证结果:', verification);
}

apiExample();
```

### 文件加密工具

```javascript
const crypto = require('crypto');
const fs = require('fs');
const path = require('path');

// 文件加密
function encryptFile(inputPath, outputPath, password) {
  const algorithm = 'aes-256-cbc';
  const key = crypto.scryptSync(password, 'salt', 32);
  const iv = crypto.randomBytes(16);

  const cipher = crypto.createCipheriv(algorithm, key, iv);
  const input = fs.createReadStream(inputPath);
  const output = fs.createWriteStream(outputPath);

  // 将 IV 写入文件开头
  output.write(iv);

  return new Promise((resolve, reject) => {
    input.pipe(cipher).pipe(output)
      .on('finish', resolve)
      .on('error', reject);
  });
}

// 文件解密
function decryptFile(inputPath, outputPath, password) {
  const algorithm = 'aes-256-cbc';
  const key = crypto.scryptSync(password, 'salt', 32);

  return new Promise((resolve, reject) => {
    // 读取 IV
    const input = fs.createReadStream(inputPath);
    let iv = Buffer.alloc(0);

    input.once('readable', () => {
      iv = input.read(16);
      const decipher = crypto.createDecipheriv(algorithm, key, iv);

      const output = fs.createWriteStream(outputPath);
      input.pipe(decipher).pipe(output)
        .on('finish', resolve)
        .on('error', reject);
    });
  });
}

// 使用示例
async function fileEncryptionExample() {
  const inputFile = 'document.txt';
  const encryptedFile = 'document.enc';
  const decryptedFile = 'document-decrypted.txt';
  const password = 'my-secret-password';

  // 创建测试文件
  fs.writeFileSync(inputFile, '这是要加密的文件内容');

  try {
    await encryptFile(inputFile, encryptedFile, password);
    console.log('文件加密完成');

    await decryptFile(encryptedFile, decryptedFile, password);
    console.log('文件解密完成');

    const originalContent = fs.readFileSync(inputFile, 'utf8');
    const decryptedContent = fs.readFileSync(decryptedFile, 'utf8');
    console.log('内容匹配:', originalContent === decryptedContent);
  } catch (error) {
    console.error('加密/解密失败:', error);
  }
}

fileEncryptionExample();
```

### JWT Token 生成和验证

```javascript
const crypto = require('crypto');

// 简化的 JWT 实现
class SimpleJWT {
  constructor(secret) {
    this.secret = secret;
  }

  encode(payload, expiresIn = '1h') {
    const header = {
      alg: 'HS256',
      typ: 'JWT'
    };

    const now = Math.floor(Date.now() / 1000);
    const exp = this.parseExpiresIn(expiresIn, now);

    const fullPayload = {
      ...payload,
      iat: now,
      exp
    };

    const headerEncoded = this.base64UrlEncode(JSON.stringify(header));
    const payloadEncoded = this.base64UrlEncode(JSON.stringify(fullPayload));
    const signature = this.sign(`${headerEncoded}.${payloadEncoded}`);

    return `${headerEncoded}.${payloadEncoded}.${signature}`;
  }

  decode(token) {
    const [headerEncoded, payloadEncoded, signature] = token.split('.');

    const expectedSignature = this.sign(`${headerEncoded}.${payloadEncoded}`);
    if (signature !== expectedSignature) {
      throw new Error('Invalid signature');
    }

    const payload = JSON.parse(this.base64UrlDecode(payloadEncoded));

    // 检查过期时间
    if (payload.exp && Math.floor(Date.now() / 1000) > payload.exp) {
      throw new Error('Token expired');
    }

    return payload;
  }

  sign(data) {
    const hmac = crypto.createHmac('sha256', this.secret);
    hmac.update(data);
    return this.base64UrlEncode(hmac.digest());
  }

  base64UrlEncode(str) {
    return Buffer.from(str)
      .toString('base64')
      .replace(/\+/g, '-')
      .replace(/\//g, '_')
      .replace(/=/g, '');
  }

  base64UrlDecode(str) {
    return Buffer.from(str + '='.repeat((4 - str.length % 4) % 4), 'base64').toString();
  }

  parseExpiresIn(expiresIn, now) {
    const match = expiresIn.match(/^(\d+)([smhd])$/);
    if (!match) throw new Error('Invalid expiresIn format');

    const value = parseInt(match[1]);
    const unit = match[2];

    const units = {
      's': 1,
      'm': 60,
      'h': 3600,
      'd': 86400
    };

    return now + (value * units[unit]);
  }
}

// 使用示例
const jwt = new SimpleJWT('my-secret-key');

// 生成 token
const token = jwt.encode({
  userId: 123,
  username: 'john.doe',
  role: 'admin'
}, '1h');

console.log('Token:', token);

// 验证 token
try {
  const decoded = jwt.decode(token);
  console.log('Decoded:', decoded);
} catch (error) {
  console.error('Token 验证失败:', error.message);
}
```

## 注意事项

1. **密钥安全**：永远不要硬编码密钥，使用环境变量或安全的密钥管理服务。

```javascript
// 推荐：从环境变量读取
const secretKey = process.env.SECRET_KEY;
```

2. **使用现代算法**：避免使用已不安全的算法（如 MD5、SHA-1、DES）。

```javascript
// 推荐
const hash = crypto.createHash('sha256');

// 不推荐
const hash = crypto.createHash('md5');
```

3. **正确处理错误**：加密操作可能会失败，必须正确处理错误。

```javascript
try {
  const encrypted = encrypt(data, key, iv);
} catch (error) {
  console.error('加密失败:', error);
  // 处理错误情况
}
```

4. **使用安全的随机数**：永远不要使用 `Math.random()` 生成加密密钥。

```javascript
// 推荐
const secureRandom = crypto.randomBytes(32);

// 不推荐
const insecureRandom = Math.random();
```

5. **密码哈希**：永远不要直接存储明文密码，使用专门的密码哈希算法。

```javascript
// 推荐
const hashedPassword = await hashPassword(password);

// 不推荐
const plaintextPassword = password; // 危险！
```

6. **时序攻击防护**：使用 `crypto.timingSafeEqual()` 比较敏感数据。

```javascript
// 安全比较
const isEqual = crypto.timingSafeEqual(
  Buffer.from(a),
  Buffer.from(b)
);

// 不安全的比较
const isUnsafeEqual = (a === b); // 可能泄露信息
```

7. **初始化向量**：每次加密都应该使用新的 IV，不要重用 IV。

```javascript
// 推荐：每次加密生成新 IV
const iv = crypto.randomBytes(16);
const cipher = crypto.createCipheriv('aes-256-cbc', key, iv);

// 不推荐：重用 IV
const sameIV = Buffer.alloc(16); // 危险！
```

8. **密钥长度**：使用足够长的密钥来保证安全性。

```javascript
// 推荐
const key = crypto.randomBytes(32); // 256 位

// 不推荐
const shortKey = crypto.randomBytes(8); // 64 位，太短
```

9. **数据完整性**：使用 HMAC 或认证加密模式（如 GCM）确保数据完整性。

10. **定期更新**：定期审查和更新加密算法和密钥长度，跟上安全最佳实践。

## 总结

Node.js 的 Crypto 模块提供了完整的加密解决方案，涵盖了哈希、对称加密、非对称加密和数字签名等功能。通过正确使用这些工具，可以实现安全的数据存储、传输和认证。在实际应用中，安全性永远是第一位的，必须严格遵循加密最佳实践，避免常见的安全漏洞。掌握 Crypto 模块的使用对于构建安全可靠的 Node.js 应用程序至关重要。