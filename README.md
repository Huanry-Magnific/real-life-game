# Real Life Game - 區塊鏈遊戲平台

## 項目概述

Real Life Game 是一個基於區塊鏈的遊戲平台，整合 ERC-20 代幣獎勵機制，讓組織者能夠創建各種遊戲活動，參與者可以透過遊戲獲得代幣獎勵。平台支援多種遊戲類型，從問答遊戲到實境闖關遊戲。

## 核心功能

### 🎮 遊戲管理
- **遊戲創建**：組織者可以創建多種類型的遊戲
- **模組化設計**：採用插件式架構，支援多種遊戲類型接入
- **首發遊戲**：問答遊戲（Q&A Game）
- **未來擴展**：實境闖關遊戲、解謎遊戲等

### 💰 獎勵機制
- **智能合約管理**：組織者將獎金存入智能合約
- **靈活分配方式**：
  - 排名制：前三名不同比例分配
  - 積分制：依照分數比例分配
  - 自定義：組織者自定義分配規則
- **自動分發**：智能合約自動執行獎金分配

### 💡 提示系統
- **加碼機制**：參與者可透過加碼獎金池獲得答案提示
- **提示管理**：組織者預設提示內容和加碼要求
- **動態定價**：根據獎金池大小調整提示價格

### 🔐 用戶系統
- **錢包登入**：支援以太坊相容錢包登入
- **身份驗證**：基於區塊鏈的去中心化身份驗證
- **用戶資料**：遊戲歷史、獲獎記錄、代幣餘額

## 技術架構

### 前端技術棧
- **框架**：Next.js 14 (App Router)
- **樣式**：Tailwind CSS
- **狀態管理**：Zustand
- **錢包整合**：WalletConnect, MetaMask
- **UI 組件**：Radix UI + shadcn/ui

### 後端技術棧
- **數據庫**：Supabase (PostgreSQL)
- **認證**：Supabase Auth + 錢包簽名驗證
- **API**：Next.js API Routes
- **文件存儲**：Supabase Storage

### 區塊鏈技術
- **網絡**：Polygon (MATIC)
- **智能合約**：Solidity
- **開發框架**：Hardhat
- **錢包整合**：ethers.js / viem
- **代幣標準**：ERC-20

## 智能合約設計

### GameFactory 合約
```solidity
// 遊戲工廠合約，管理遊戲創建和註冊
contract GameFactory {
    mapping(uint256 => address) public games;
    mapping(address => bool) public authorizedGameTypes;
    
    function createGame(address gameType, bytes calldata initData) external;
    function registerGameType(address gameType) external;
}
```

### GameReward 合約
```solidity
// 獎勵分配合約
contract GameReward {
    struct RewardPool {
        address token;
        uint256 totalAmount;
        uint256 remainingAmount;
        RewardDistribution distribution;
    }
    
    enum RewardDistribution {
        RANKING,    // 排名制
        SCORE,      // 積分制
        CUSTOM      // 自定義
    }
    
    function depositReward(uint256 gameId, address token, uint256 amount) external;
    function distributeRewards(uint256 gameId, address[] calldata winners, uint256[] calldata amounts) external;
}
```

### QAGame 合約
```solidity
// 問答遊戲合約
contract QAGame {
    struct Question {
        string questionHash;  // IPFS hash
        bytes32 answerHash;   // 答案哈希
        uint256 hintPrice;    // 提示價格
        string hintHash;      // 提示內容哈希
    }
    
    function submitAnswer(uint256 questionId, string calldata answer) external;
    function buyHint(uint256 questionId) external payable;
    function endGame() external;
}
```

## 數據庫設計

### 核心表結構

```sql
-- 用戶表
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address TEXT UNIQUE NOT NULL,
    username TEXT,
    avatar_url TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 遊戲表
CREATE TABLE games (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    contract_address TEXT UNIQUE NOT NULL,
    creator_id UUID REFERENCES users(id),
    game_type TEXT NOT NULL,
    title TEXT NOT NULL,
    description TEXT,
    status TEXT DEFAULT 'created',
    start_time TIMESTAMP WITH TIME ZONE,
    end_time TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 問答遊戲問題表
CREATE TABLE qa_questions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    game_id UUID REFERENCES games(id),
    question_text TEXT NOT NULL,
    answer_hash TEXT NOT NULL,
    hint_text TEXT,
    hint_price DECIMAL,
    order_index INTEGER,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 遊戲參與記錄
CREATE TABLE game_participations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    game_id UUID REFERENCES games(id),
    user_id UUID REFERENCES users(id),
    score INTEGER DEFAULT 0,
    rank INTEGER,
    reward_amount DECIMAL,
    completed_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 答題記錄
CREATE TABLE answer_submissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    participation_id UUID REFERENCES game_participations(id),
    question_id UUID REFERENCES qa_questions(id),
    answer_text TEXT NOT NULL,
    is_correct BOOLEAN,
    submitted_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

## 項目結構

```
real-life-game/
├── README.md
├── package.json
├── next.config.js
├── tailwind.config.js
├── tsconfig.json
├── .env.local
├── contracts/                 # 智能合約
│   ├── GameFactory.sol
│   ├── GameReward.sol
│   ├── QAGame.sol
│   └── deploy/
├── src/
│   ├── app/                  # Next.js App Router
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── games/
│   │   ├── create/
│   │   └── api/
│   ├── components/           # React 組件
│   │   ├── ui/              # 基礎 UI 組件
│   │   ├── game/            # 遊戲相關組件
│   │   ├── wallet/          # 錢包組件
│   │   └── layout/          # 布局組件
│   ├── lib/                 # 工具函數
│   │   ├── supabase.ts
│   │   ├── contracts.ts
│   │   ├── wallet.ts
│   │   └── utils.ts
│   ├── hooks/               # React Hooks
│   ├── store/               # 狀態管理
│   └── types/               # TypeScript 類型
├── supabase/                # Supabase 配置
│   ├── migrations/
│   └── seed.sql
└── hardhat.config.js        # Hardhat 配置
```

## 開發路線圖

### Phase 1: 基礎架構 (4週)
- [x] 項目初始化和技術棧搭建
- [ ] 智能合約開發和測試
- [ ] 數據庫設計和 Supabase 配置
- [ ] 錢包連接和用戶認證
- [ ] 基礎 UI 組件開發

### Phase 2: 問答遊戲 (6週)
- [ ] 問答遊戲智能合約
- [ ] 遊戲創建和管理界面
- [ ] 問答遊戲參與界面
- [ ] 提示系統實現
- [ ] 獎勵分配機制

### Phase 3: 優化和擴展 (4週)
- [ ] 性能優化和安全審計
- [ ] 移動端適配
- [ ] 遊戲數據分析
- [ ] 社交功能（排行榜、分享）

### Phase 4: 實境遊戲 (8週)
- [ ] 實境遊戲框架設計
- [ ] GPS 定位和 AR 整合
- [ ] 闖關系統開發
- [ ] 多人協作機制

## 安全考量

### 智能合約安全
- **重入攻擊防護**：使用 ReentrancyGuard
- **權限控制**：基於角色的訪問控制
- **代碼審計**：第三方安全審計
- **升級機制**：代理合約模式

### 前端安全
- **輸入驗證**：所有用戶輸入嚴格驗證
- **XSS 防護**：內容安全策略 (CSP)
- **私鑰安全**：永不接觸用戶私鑰
- **HTTPS**：全站 HTTPS 加密

### 數據安全
- **敏感數據加密**：答案和提示加密存儲
- **訪問控制**：基於 RLS 的數據訪問控制
- **備份策略**：定期數據備份
- **隱私保護**：符合 GDPR 要求

## 部署和運維

### 開發環境
```bash
# 安裝依賴
npm install

# 啟動開發服務器
npm run dev

# 編譯智能合約
npx hardhat compile

# 運行測試
npm run test
```

### 生產環境
- **前端部署**：Vercel
- **數據庫**：Supabase Cloud
- **智能合約**：Polygon Mainnet
- **監控**：Sentry + Datadog
- **CDN**：Cloudflare

## 貢獻指南

1. Fork 項目
2. 創建功能分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 開啟 Pull Request

## 許可證

本項目採用 MIT 許可證 - 查看 [LICENSE](LICENSE) 文件了解詳情。

## 聯繫方式

- 項目維護者：[Your Name]
- 電子郵件：[your.email@example.com]
- 項目鏈接：[https://github.com/yourusername/real-life-game](https://github.com/yourusername/real-life-game)

---

**注意**：本項目仍在開發中，功能和 API 可能會發生變化。請關注項目更新和發布說明。