* MongoDB 官网地址：https://www.mongodb.com/
* MongoDB 官方英文文档：https://docs.mongodb.com/manual/
* MongoDB 各平台下载地址：https://www.mongodb.com/try/download/community





# 一、MongoDB开发指南

MongoDB是一个流行的NoSQL数据库，它使用文档存储模型（类似JSON格式）来存储数据。以下是关于如何开发和使用MongoDB的详细指南：

## 1. 基础安装与启动

### 在本地安装MongoDB

1. **macOS** (使用Homebrew):
   ```bash
   brew tap mongodb/brew
   brew install mongodb-community
   ```

2. **启动MongoDB服务**:
   ```bash
   # 使用Homebrew启动服务
   brew services start mongodb-community
   
   # 或手动启动
   mongod --dbpath=/data/db
   ```

3. **验证服务是否运行**:
   ```bash
   # 查看MongoDB进程
   ps aux | grep -v grep | grep mongod
   
   # 或连接到MongoDB shell
   mongosh
   ```

## 2.MongoDB与Node.js开发

### 1. 安装必要的依赖

```bash
npm install mongoose
# 或
pnpm add mongoose
# 或
yarn add mongoose
```

### 2. 创建数据库连接

```javascript
const mongoose = require('mongoose');

// 连接MongoDB
mongoose.connect('mongodb://localhost:27017/your-database-name', {
  // 从MongoDB 4.0起，这些选项已不再需要，但保留它们不会有害
  useNewUrlParser: true,
  useUnifiedTopology: true
})
.then(() => console.log('MongoDB连接成功'))
.catch(err => console.error('MongoDB连接错误:', err));
```

### 3. 定义模型(Schema)

```javascript
const mongoose = require('mongoose');

// 定义模式
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true,
    trim: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    match: [/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/, '请提供有效的邮箱']
  },
  password: {
    type: String,
    required: true,
    minlength: 6,
    select: false  // 默认查询不返回密码
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// 创建模型
const User = mongoose.model('User', userSchema);

module.exports = User;
```

### 4. 基本CRUD操作

```javascript
// 创建新文档
const createUser = async (userData) => {
  try {
    const user = await User.create(userData);
    return user;
  } catch (error) {
    throw new Error(`创建用户失败: ${error.message}`);
  }
};

// 查询文档
const getUsers = async () => {
  try {
    const users = await User.find();
    return users;
  } catch (error) {
    throw new Error(`获取用户列表失败: ${error.message}`);
  }
};

// 查询单个文档
const getUserById = async (id) => {
  try {
    const user = await User.findById(id);
    if (!user) {
      throw new Error('未找到该用户');
    }
    return user;
  } catch (error) {
    throw new Error(`获取用户失败: ${error.message}`);
  }
};

// 更新文档
const updateUser = async (id, updateData) => {
  try {
    const user = await User.findByIdAndUpdate(id, updateData, {
      new: true,        // 返回更新后的文档
      runValidators: true // 运行验证器
    });
    if (!user) {
      throw new Error('未找到该用户');
    }
    return user;
  } catch (error) {
    throw new Error(`更新用户失败: ${error.message}`);
  }
};

// 删除文档
const deleteUser = async (id) => {
  try {
    const user = await User.findByIdAndDelete(id);
    if (!user) {
      throw new Error('未找到该用户');
    }
    return user;
  } catch (error) {
    throw new Error(`删除用户失败: ${error.message}`);
  }
};
```

## 3. 高级查询和技巧

### 1. 高级查询

```javascript
// 条件查询
const findActiveUsers = async () => {
  return await User.find({ isActive: true });
};

// 排序
const getUsersSortedByDate = async () => {
  return await User.find().sort({ createdAt: -1 }); // -1降序，1升序
};

// 限制结果数量
const getRecentUsers = async () => {
  return await User.find().sort({ createdAt: -1 }).limit(10);
};

// 分页查询
const getPaginatedUsers = async (page = 1, limit = 10) => {
  const skip = (page - 1) * limit;
  return await User.find().skip(skip).limit(limit);
};

// 使用正则表达式查询
const searchUsersByName = async (keyword) => {
  return await User.find({
    username: { $regex: keyword, $options: 'i' } // i表示不区分大小写
  });
};
```

### 2. 数据聚合

```javascript
const getAggregatedData = async () => {
  return await User.aggregate([
    { $match: { isActive: true } },  // 筛选条件
    { $group: {                     // 分组条件
        _id: '$role',               // 按角色分组
        count: { $sum: 1 },         // 计数
        avgAge: { $avg: '$age' }    // 计算平均年龄
      }
    },
    { $sort: { count: -1 } }        // 按计数降序排序
  ]);
};
```

### 3. 使用索引提高性能

```javascript
// 在Schema中定义索引
userSchema.index({ username: 1 });  // 1表示升序索引
userSchema.index({ email: 1 });

// 或者手动创建索引
User.createIndexes();
```

## 4. 生产环境的最佳实践

### 1. 使用环境变量存储敏感信息

```javascript
// 使用dotenv包加载环境变量
require('dotenv').config();

mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true
});
```

### 2. 连接池配置

```javascript
mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  poolSize: 10  // 设置连接池大小
});
```

### 3. 错误处理

```javascript
// 全局错误处理
mongoose.connection.on('error', (err) => {
  console.error('MongoDB连接错误:', err);
  // 适当的重试逻辑或退出应用
});

mongoose.connection.on('disconnected', () => {
  console.log('MongoDB断开连接');
  // 重连逻辑
});

// 进程结束时关闭连接
process.on('SIGINT', async () => {
  await mongoose.connection.close();
  console.log('MongoDB连接已关闭');
  process.exit(0);
});
```

### 4. 使用MongoDB Atlas云服务

如果您不想在本地维护MongoDB服务器，可以使用MongoDB Atlas提供的云数据库服务：

1. 注册并登录[MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. 创建一个集群（有免费层可用）
3. 设置数据库访问权限和网络访问
4. 获取连接字符串并在应用中使用

连接字符串例子：
```
mongodb+srv://<username>:<password>@cluster0.mongodb.net/<dbname>?retryWrites=true&w=majority
```

## 5. 调试与工具

1. **MongoDB Compass** - 图形化界面工具，用于查看和管理数据库
2. **Studio 3T** - 另一个强大的MongoDB管理工具
3. **mongoose-debug** - 调试Mongoose查询的工具

## 6. 安全最佳实践

1. 使用认证保护数据库
2. 始终对敏感数据进行加密（例如使用bcrypt加密密码）
3. 实施适当的权限控制
4. 定期进行数据备份

通过遵循这些指南，您可以有效地使用MongoDB进行开发，并构建出高效、可靠的应用程序。希望这些信息对您有所帮助！