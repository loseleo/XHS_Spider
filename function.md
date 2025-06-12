# 小红书爬虫 SDK

一个用于爬取小红书笔记内容的 Node.js SDK，支持获取笔记标题、内容、无水印图片、作者信息等。

## 特性

- 支持小红书笔记内容爬取
- 支持无水印图片获取
- 内置 Cookie 管理功能
- 自动处理签名算法
- 支持短链接解析
- 支持获取用户信息和笔记列表
- 支持搜索笔记和获取推荐内容
- 支持创作者笔记获取

## 安装

```bash
npm install xhs-crawler-sdk
```

## 基本使用

### 爬取笔记

```typescript
import { crawlNote } from 'xhs-crawler-sdk';

async function example() {
  try {
    // 爬取笔记
    const result = await crawlNote('https://www.xiaohongshu.com/explore/你的笔记ID');
    
    console.log('标题:', result.title);
    console.log('内容:', result.content);
    console.log('图片:', result.images);
    console.log('作者:', result.author?.name);
    console.log('点赞数:', result.stats?.likeCount);
  } catch (error) {
    console.error('爬取失败:', error);
  }
}

example();
```

### 使用自定义 Cookie

```typescript
import { crawlNote } from 'xhs-crawler-sdk';

async function example() {
  const customCookie = '你的小红书Cookie字符串';
  
  try {
    const result = await crawlNote('https://www.xiaohongshu.com/explore/你的笔记ID', customCookie);
    console.log('爬取结果:', result);
  } catch (error) {
    console.error('爬取失败:', error);
  }
}

example();
```

## API 接口

### 获取用户信息

```typescript
import { getUserInfo } from 'xhs-crawler-sdk';

async function example() {
  try {
    // 用户ID可以从用户主页URL中获取
    const userId = '5ff0e6a3000000000101d84e';
    const result = await getUserInfo(userId);
    
    if (result.success && result.user) {
      console.log('用户昵称:', result.user.nickname);
      console.log('粉丝数:', result.user.fans);
      console.log('笔记数:', result.user.notes);
    }
  } catch (error) {
    console.error('获取用户信息失败:', error);
  }
}
```

### 获取用户笔记

```typescript
import { getUserNotes } from 'xhs-crawler-sdk';

async function example() {
  try {
    const userId = '5ff0e6a3000000000101d84e';
    const maxNotes = 10; // 最多获取10条笔记
    
    const result = await getUserNotes(userId, undefined, maxNotes);
    
    if (result.success && result.notes.length > 0) {
      console.log(`获取到 ${result.notes.length} 条笔记`);
      
      // 打印笔记标题列表
      result.notes.forEach((note, index) => {
        console.log(`笔记 ${index + 1}:`, {
          title: note.display_title || '无标题',
          noteId: note.note_id,
          likes: note.interact_info?.liked_count || 0
        });
      });
    }
  } catch (error) {
    console.error('获取用户笔记失败:', error);
  }
}
```

### 搜索笔记

```typescript
import { searchNotes } from 'xhs-crawler-sdk';

async function example() {
  try {
    const keyword = '旅行';
    const page = 1; // 第一页
    const sort = 'popularity_descending'; // 按热度排序，可选值: general(综合), popularity_descending(热度), time_descending(时间)
    
    const result = await searchNotes(keyword, page, sort);
    
    if (result.success && result.notes.length > 0) {
      console.log(`搜索到 ${result.notes.length} 条笔记`);
      console.log('是否有更多:', result.hasMore);
      
      // 打印搜索结果
      result.notes.forEach((note, index) => {
        console.log(`搜索结果 ${index + 1}:`, {
          title: note.note_card?.display_title || '无标题',
          noteId: note.note_card?.note_id
        });
      });
    }
  } catch (error) {
    console.error('搜索笔记失败:', error);
  }
}
```

### 获取推荐笔记

```typescript
import { getRecommendNotes } from 'xhs-crawler-sdk';

async function example() {
  try {
    const category = ''; // 空字符串表示默认推荐
    const requireNum = 5; // 获取5条推荐笔记
    
    const result = await getRecommendNotes(category, requireNum);
    
    if (result.success && result.notes.length > 0) {
      console.log(`获取到 ${result.notes.length} 条推荐笔记`);
      
      // 打印推荐笔记
      result.notes.forEach((note, index) => {
        console.log(`推荐笔记 ${index + 1}:`, {
          title: note.note_card?.display_title || '无标题',
          noteId: note.note_card?.note_id
        });
      });
    }
  } catch (error) {
    console.error('获取推荐笔记失败:', error);
  }
}
```

### 获取创作者笔记

```typescript
import { getCreatorNotes } from 'xhs-crawler-sdk';

async function example() {
  try {
    // 注意：此功能需要创作者Cookie
    const result = await getCreatorNotes();
    
    if (result.success && result.notes.length > 0) {
      console.log(`获取到 ${result.notes.length} 条创作者笔记`);
      
      // 打印创作者笔记
      result.notes.forEach((note, index) => {
        console.log(`创作者笔记 ${index + 1}:`, {
          title: note.title || '无标题',
          noteId: note.note_id,
          likes: note.liked_count || 0,
          comments: note.comment_count || 0
        });
      });
    }
  } catch (error) {
    console.error('获取创作者笔记失败:', error);
  }
}
```

## Cookie 管理

### 添加和管理 Cookie

```typescript
import { cookieService } from 'xhs-crawler-sdk';

async function manageCookies() {
  // 添加 Cookie
  const cookie1 = await cookieService.addCookie('我的Cookie1', '你的Cookie字符串', 10);
  const cookie2 = await cookieService.addCookie('我的Cookie2', '另一个Cookie字符串', 5);
  
  // 获取所有 Cookie
  const allCookies = cookieService.getAllCookies();
  console.log('所有Cookie:', allCookies);
  
  // 获取 Cookie 状态
  const status = cookieService.getCookieStatus();
  console.log('Cookie状态:', status);
  console.log('状态消息:', cookieService.getExpiryMessage(status));
  
  // 验证特定 Cookie
  const isValid = await cookieService.validateCookie(cookie1);
  console.log('Cookie有效性:', isValid);
  
  // 重新检查所有 Cookie
  const checkResult = await cookieService.recheckAllCookies();
  console.log('检查结果:', checkResult);
}

manageCookies();
```

## 配置

### 自定义配置

```typescript
import { updateConfig, XHS_CONFIG } from 'xhs-crawler-sdk';

// 更新配置
updateConfig({
  REQUEST_TIMEOUT: 30000, // 请求超时时间，单位毫秒
  MAX_RETRY_COUNT: 5,     // 最大重试次数
  CHECK_INTERVAL: 3600000, // Cookie 检查间隔，单位毫秒（1小时）
});

console.log('当前配置:', XHS_CONFIG);
```

## 日志

### 自定义日志级别

```typescript
import { log, LogLevel } from 'xhs-crawler-sdk';

// 设置日志级别
log.setLevel(LogLevel.DEBUG); // 可选值: DEBUG, INFO, WARN, ERROR

// 设置日志前缀
log.setPrefix('[MyApp]');
```

## 注意事项

1. 请合理使用爬虫，遵守网站的使用条款和robots.txt规则
2. Cookie 可能会过期，需要定期更新
3. 签名算法可能会随着小红书官方更新而失效，请保持SDK版本更新

## 许可证

MIT
