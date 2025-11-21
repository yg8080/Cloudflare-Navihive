
# NaviHive - 现代化个人导航站

![NaviHive 导航站](https://img.shields.io/badge/NaviHive-导航站-blue)
![React](https://img.shields.io/badge/React-19.0.0-61dafb)
![TypeScript](https://img.shields.io/badge/TypeScript-5.7-3178c6)
![Material UI](https://img.shields.io/badge/Material_UI-7.0-0081cb)
![Cloudflare](https://img.shields.io/badge/Cloudflare-Workers-f38020)
![License](https://img.shields.io/badge/License-MIT-green)

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/YG8080/Cloudflare-Navihive)


#### 第一步：创建 D1 数据库

1. 登录 [Cloudflare 控制台](https://dash.cloudflare.com/)
2. 在左侧菜单选择 **Workers & Pages**
3. 点击右侧的 **D1 SQL Database** 标签
4. 点击 **Create database** 按钮
5. 输入数据库名称：`navigation-db`
6. 点击 **Create** 完成创建
7. **记下数据库 ID**（格式类似：`43ff28e1-42d6-4e53-9657-0702ae1353b6`）


#### 第二步：配置项目

1. 在 Cloudflare 控制台，进入 **Workers & Pages**
2. 如果已通过一键部署创建了 Worker，选择该项目；否则点击 **Create Application** > **Create Worker**
3. 进入项目后，点击 **Settings** > **Variables**
4. 添加以下环境变量：

| 变量名 | 值 | 说明 |
|--------|-----|------|
| `AUTH_ENABLED` | `true` | 启用登录认证 |
| `AUTH_REQUIRED_FOR_READ` | `false` | 是否要求访客认证。`false` 启用免登陆只读模式 |
| `AUTH_USERNAME` | `admin` | 管理员用户名（可自定义） |
| `AUTH_PASSWORD` | 见下方说明 | 管理员密码（bcrypt 哈希） |
| `AUTH_SECRET` | 见下方说明 | JWT 密钥（随机字符串） |

> 如果需要完全关闭访客访问，将 `AUTH_REQUIRED_FOR_READ` 设置为 `true` 并重新部署；保留默认的 `false` 即表示启用公开的只读模式。

**设置密码（重要）：**
- 访问 [bcrypt 在线生成器](https://bcrypt.online/) 或使用 [Bcrypt Generator](https://bcrypt-generator.com/)
- 输入你想要的密码，选择 **10 rounds**
- 复制生成的哈希值（格式类似：`$2b$10$abc123...`）
- 粘贴到 `AUTH_PASSWORD` 变量中

**设置 JWT 密钥：**
- 访问 [随机字符串生成器](https://www.random.org/strings/)
- 生成一个 32 位以上的随机字符串
- 粘贴到 `AUTH_SECRET` 变量中

5. 点击 **Settings** > **Bindings** > **Add binding**
6. 选择 **D1 database**，Variable name 填写 `DB`，选择刚才创建的 `navigation-db`

#### 第三步：初始化数据库

1. 在 Cloudflare 控制台，进入 **D1 Databases**
2. 点击 `navigation-db` 数据库
3. 选择 **Console** 标签
4. 复制以下 SQL 命令并执行（点击 **Execute**）：

```sql
-- 创建分组表
CREATE TABLE IF NOT EXISTS groups (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    order_num INTEGER NOT NULL,
    is_public INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 创建站点表
CREATE TABLE IF NOT EXISTS sites (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    group_id INTEGER NOT NULL,
    name TEXT NOT NULL,
    url TEXT NOT NULL,
    icon TEXT,
    description TEXT,
    notes TEXT,
    order_num INTEGER NOT NULL,
    is_public INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (group_id) REFERENCES groups(id) ON DELETE CASCADE
);

-- 创建配置表
CREATE TABLE IF NOT EXISTS configs (
    key TEXT PRIMARY KEY,
    value TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 标记数据库已初始化
INSERT INTO configs (key, value) VALUES ('DB_INITIALIZED', 'true');

-- 创建只读模式所需索引
CREATE INDEX IF NOT EXISTS idx_groups_is_public ON groups(is_public);
CREATE INDEX IF NOT EXISTS idx_sites_is_public ON sites(is_public);
```



5. 确认每条命令都执行成功（显示 ✓ Success）

> 如果你已经运行过旧版本的 SQL，只需在数据库控制台执行 `migrations/002_add_is_public.sql` 中的语句，或在本地使用 `wrangler d1 execute navigation-db --file=migrations/002_add_is_public.sql` 即可完成字段升级。

