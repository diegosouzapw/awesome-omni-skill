---
name: sage-workspace-detection
description: Sage 工作区检测开发指南，涵盖项目类型检测、语言识别、依赖分析、Git 信息
when_to_use: 当涉及项目结构分析、语言检测、依赖解析、工作区信息获取时使用
allowed_tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
  - Bash
user_invocable: true
priority: 88
---

# Sage 工作区检测开发指南

## 模块概览

工作区模块提供项目分析和类型检测功能，代码量 **3123 行**，包含：

```
crates/sage-core/src/workspace/
├── mod.rs              # 公开接口 (32行)
├── analyzer.rs         # WorkspaceAnalyzer
├── models.rs           # 数据模型 (227行)
├── structure.rs        # 项目结构 (97行)
├── statistics.rs       # 文件统计 (165行)
├── entry_points.rs     # 入口点检测 (73行)
├── git.rs              # Git 信息 (43行)
├── detector/           # 项目类型检测
│   ├── mod.rs          # 入口
│   ├── language_detection.rs # 语言检测
│   ├── framework_detection.rs # 框架检测
│   ├── types/          # 类型定义
│   │   ├── mod.rs      # 入口
│   │   ├── language.rs # LanguageType (70行)
│   │   ├── framework.rs # FrameworkType
│   │   ├── build_system.rs # BuildSystem
│   │   ├── runtime.rs  # RuntimeType (36行)
│   │   └── test_framework.rs # TestFramework (50行)
│   └── detectors/      # 检测器实现
│       ├── mod.rs      # 入口
│       ├── rust.rs     # Rust 检测
│       ├── python.rs   # Python 检测
│       ├── node.rs     # Node.js 检测
│       ├── go.rs       # Go 检测
│       ├── jvm.rs      # JVM 检测
│       └── other.rs    # 其他语言
├── dependencies/       # 依赖分析
│   ├── mod.rs          # 入口
│   ├── cargo.rs        # Rust Cargo
│   ├── npm.rs          # Node.js NPM
│   ├── python.rs       # Python
│   └── go.rs           # Go Modules
└── patterns/           # 模式匹配
    ├── mod.rs          # 入口 (164行)
    ├── matcher.rs      # PatternMatcher (133行)
    ├── language_patterns.rs # 语言模式 (313行)
    └── types.rs        # 类型 (115行)
```

---

## 一、核心架构

### 1.1 WorkspaceAnalyzer

```rust
// crates/sage-core/src/workspace/analyzer.rs
pub struct WorkspaceAnalyzer {
    config: WorkspaceConfig,
    detector: ProjectTypeDetector,
    pattern_matcher: PatternMatcher,
}

impl WorkspaceAnalyzer {
    /// 分析工作区
    pub async fn analyze(&self, path: &Path) -> Result<AnalysisResult, WorkspaceError> {
        // 1. 检测项目类型
        let project_type = self.detector.detect(path)?;

        // 2. 获取文件统计
        let file_stats = self.collect_file_stats(path)?;

        // 3. 查找重要文件
        let important_files = self.pattern_matcher.find_important_files(path)?;

        // 4. 分析依赖
        let dependencies = if self.config.analyze_dependencies {
            self.analyze_dependencies(path, &project_type)?
        } else {
            None
        };

        // 5. 获取 Git 信息
        let git_info = if self.config.analyze_git {
            git::get_git_info(path).ok()
        } else {
            None
        };

        // 6. 检测入口点
        let entry_points = entry_points::detect_entry_points(path, &project_type)?;

        Ok(AnalysisResult {
            project_type,
            file_stats,
            important_files,
            dependencies,
            git_info,
            entry_points,
            structure: self.analyze_structure(path)?,
        })
    }
}
```

### 1.2 分析结果

```rust
// crates/sage-core/src/workspace/models.rs
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct AnalysisResult {
    /// 项目类型
    pub project_type: ProjectType,
    /// 文件统计
    pub file_stats: FileStats,
    /// 重要文件
    pub important_files: Vec<ImportantFile>,
    /// 依赖信息
    pub dependencies: Option<DependencyInfo>,
    /// Git 信息
    pub git_info: Option<GitInfo>,
    /// 入口点
    pub entry_points: Vec<EntryPoint>,
    /// 项目结构
    pub structure: ProjectStructure,
}
```

---

## 二、项目类型检测

### 2.1 ProjectType

```rust
// crates/sage-core/src/workspace/detector/types/
#[derive(Debug, Clone)]
pub struct ProjectType {
    /// 主语言
    pub primary_language: LanguageType,
    /// 其他语言
    pub secondary_languages: Vec<LanguageType>,
    /// 框架
    pub frameworks: Vec<FrameworkType>,
    /// 构建系统
    pub build_systems: Vec<BuildSystem>,
    /// 运行时
    pub runtime: Option<RuntimeType>,
    /// 测试框架
    pub test_frameworks: Vec<TestFramework>,
}
```

### 2.2 语言类型

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub enum LanguageType {
    Rust,
    Python,
    JavaScript,
    TypeScript,
    Go,
    Java,
    Kotlin,
    Scala,
    Ruby,
    PHP,
    CSharp,
    CPlusPlus,
    C,
    Swift,
    Dart,
    Elixir,
    Haskell,
    Unknown,
}

impl LanguageType {
    /// 获取语言名称
    pub fn name(&self) -> &'static str {
        match self {
            Self::Rust => "Rust",
            Self::Python => "Python",
            Self::JavaScript => "JavaScript",
            Self::TypeScript => "TypeScript",
            // ...
        }
    }

    /// 获取文件扩展名
    pub fn extensions(&self) -> &[&str] {
        match self {
            Self::Rust => &["rs"],
            Self::Python => &["py", "pyi"],
            Self::JavaScript => &["js", "mjs", "cjs"],
            Self::TypeScript => &["ts", "tsx", "mts"],
            // ...
        }
    }
}
```

### 2.3 框架类型

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum FrameworkType {
    // Web 框架
    React,
    Vue,
    Angular,
    Svelte,
    NextJs,
    NuxtJs,
    // 后端框架
    Express,
    Fastify,
    NestJs,
    Django,
    Flask,
    FastApi,
    Actix,
    Rocket,
    Axum,
    Gin,
    Echo,
    Spring,
    // 移动
    ReactNative,
    Flutter,
    // 其他
    Electron,
    Tauri,
}
```

### 2.4 构建系统

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum BuildSystem {
    Cargo,       // Rust
    Npm,         // Node.js
    Yarn,        // Node.js
    Pnpm,        // Node.js
    Pip,         // Python
    Poetry,      // Python
    Pipenv,      // Python
    Uv,          // Python
    GoMod,       // Go
    Maven,       // Java
    Gradle,      // JVM
    CMake,       // C/C++
    Make,        // 通用
    Bazel,       // 通用
}
```

---

## 三、ProjectTypeDetector

### 3.1 检测流程

```rust
// crates/sage-core/src/workspace/detector/mod.rs
pub struct ProjectTypeDetector {
    detectors: Vec<Box<dyn LanguageDetector>>,
}

impl ProjectTypeDetector {
    pub fn new() -> Self {
        Self {
            detectors: vec![
                Box::new(RustDetector),
                Box::new(PythonDetector),
                Box::new(NodeDetector),
                Box::new(GoDetector),
                Box::new(JvmDetector),
            ],
        }
    }

    /// 检测项目类型
    pub fn detect(&self, path: &Path) -> Result<ProjectType, WorkspaceError> {
        let mut languages = Vec::new();
        let mut frameworks = Vec::new();
        let mut build_systems = Vec::new();

        for detector in &self.detectors {
            if let Some(result) = detector.detect(path)? {
                languages.push(result.language);
                frameworks.extend(result.frameworks);
                build_systems.extend(result.build_systems);
            }
        }

        // 确定主语言
        let primary = self.determine_primary_language(&languages, path);

        Ok(ProjectType {
            primary_language: primary,
            secondary_languages: languages.into_iter().filter(|l| *l != primary).collect(),
            frameworks,
            build_systems,
            runtime: self.detect_runtime(path)?,
            test_frameworks: self.detect_test_frameworks(path)?,
        })
    }
}
```

### 3.2 语言检测器 Trait

```rust
pub trait LanguageDetector: Send + Sync {
    /// 检测语言
    fn detect(&self, path: &Path) -> Result<Option<DetectionResult>, WorkspaceError>;

    /// 支持的语言
    fn language(&self) -> LanguageType;
}

#[derive(Debug)]
pub struct DetectionResult {
    pub language: LanguageType,
    pub frameworks: Vec<FrameworkType>,
    pub build_systems: Vec<BuildSystem>,
    pub confidence: f32,
}
```

### 3.3 Rust 检测器示例

```rust
// crates/sage-core/src/workspace/detector/detectors/rust.rs
pub struct RustDetector;

impl LanguageDetector for RustDetector {
    fn detect(&self, path: &Path) -> Result<Option<DetectionResult>, WorkspaceError> {
        let cargo_toml = path.join("Cargo.toml");

        if !cargo_toml.exists() {
            return Ok(None);
        }

        let content = fs::read_to_string(&cargo_toml)?;
        let mut frameworks = Vec::new();
        let build_systems = vec![BuildSystem::Cargo];

        // 检测框架
        if content.contains("actix-web") {
            frameworks.push(FrameworkType::Actix);
        }
        if content.contains("rocket") {
            frameworks.push(FrameworkType::Rocket);
        }
        if content.contains("axum") {
            frameworks.push(FrameworkType::Axum);
        }
        if content.contains("tauri") {
            frameworks.push(FrameworkType::Tauri);
        }

        Ok(Some(DetectionResult {
            language: LanguageType::Rust,
            frameworks,
            build_systems,
            confidence: 1.0,
        }))
    }

    fn language(&self) -> LanguageType {
        LanguageType::Rust
    }
}
```

---

## 四、依赖分析

### 4.1 DependencyInfo

```rust
// crates/sage-core/src/workspace/models.rs
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct DependencyInfo {
    /// 直接依赖
    pub direct: Vec<Dependency>,
    /// 开发依赖
    pub dev: Vec<Dependency>,
    /// 可选依赖
    pub optional: Vec<Dependency>,
    /// 依赖总数
    pub total_count: usize,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Dependency {
    pub name: String,
    pub version: Option<String>,
    pub source: DependencySource,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum DependencySource {
    Registry(String),  // 如 crates.io, npm
    Git(String),       // Git URL
    Path(PathBuf),     // 本地路径
}
```

### 4.2 Cargo 依赖分析

```rust
// crates/sage-core/src/workspace/dependencies/cargo.rs
pub fn analyze_cargo_dependencies(path: &Path) -> Result<DependencyInfo, WorkspaceError> {
    let cargo_toml = path.join("Cargo.toml");
    let content = fs::read_to_string(&cargo_toml)?;
    let manifest: toml::Value = toml::from_str(&content)?;

    let mut direct = Vec::new();
    let mut dev = Vec::new();

    // 解析 [dependencies]
    if let Some(deps) = manifest.get("dependencies").and_then(|d| d.as_table()) {
        for (name, spec) in deps {
            direct.push(parse_cargo_dependency(name, spec));
        }
    }

    // 解析 [dev-dependencies]
    if let Some(deps) = manifest.get("dev-dependencies").and_then(|d| d.as_table()) {
        for (name, spec) in deps {
            dev.push(parse_cargo_dependency(name, spec));
        }
    }

    Ok(DependencyInfo {
        total_count: direct.len() + dev.len(),
        direct,
        dev,
        optional: Vec::new(),
    })
}
```

---

## 五、模式匹配

### 5.1 PatternMatcher

```rust
// crates/sage-core/src/workspace/patterns/matcher.rs
pub struct PatternMatcher {
    patterns: Vec<ProjectPattern>,
}

impl PatternMatcher {
    /// 查找重要文件
    pub fn find_important_files(&self, path: &Path) -> Result<Vec<ImportantFile>, WorkspaceError> {
        let mut results = Vec::new();

        for pattern in &self.patterns {
            for file_pattern in &pattern.files {
                let glob = glob::glob(&format!("{}/{}", path.display(), file_pattern))?;

                for entry in glob.flatten() {
                    results.push(ImportantFile {
                        path: entry.clone(),
                        file_type: pattern.file_type,
                        description: pattern.description.clone(),
                    });
                }
            }
        }

        Ok(results)
    }
}
```

### 5.2 重要文件类型

```rust
// crates/sage-core/src/workspace/patterns/types.rs
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum ImportantFileType {
    /// 配置文件
    Config,
    /// 构建文件
    Build,
    /// 依赖声明
    Dependencies,
    /// 文档
    Documentation,
    /// 入口点
    EntryPoint,
    /// 测试
    Test,
    /// CI/CD
    Ci,
    /// Docker
    Docker,
    /// 环境变量
    Environment,
}

#[derive(Debug, Clone)]
pub struct ImportantFile {
    pub path: PathBuf,
    pub file_type: ImportantFileType,
    pub description: String,
}
```

### 5.3 语言模式定义

```rust
// crates/sage-core/src/workspace/patterns/language_patterns.rs
pub fn get_rust_patterns() -> Vec<ProjectPattern> {
    vec![
        ProjectPattern {
            file_type: ImportantFileType::Build,
            files: vec!["Cargo.toml", "Cargo.lock"],
            description: "Rust package manifest".to_string(),
        },
        ProjectPattern {
            file_type: ImportantFileType::EntryPoint,
            files: vec!["src/main.rs", "src/lib.rs"],
            description: "Rust entry point".to_string(),
        },
        ProjectPattern {
            file_type: ImportantFileType::Config,
            files: vec![".cargo/config.toml", "rust-toolchain.toml"],
            description: "Rust configuration".to_string(),
        },
    ]
}

pub fn get_node_patterns() -> Vec<ProjectPattern> {
    vec![
        ProjectPattern {
            file_type: ImportantFileType::Dependencies,
            files: vec!["package.json", "package-lock.json", "yarn.lock", "pnpm-lock.yaml"],
            description: "Node.js package manifest".to_string(),
        },
        ProjectPattern {
            file_type: ImportantFileType::Config,
            files: vec!["tsconfig.json", ".eslintrc*", ".prettierrc*"],
            description: "Node.js configuration".to_string(),
        },
    ]
}
```

---

## 六、Git 信息

```rust
// crates/sage-core/src/workspace/git.rs
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct GitInfo {
    /// 当前分支
    pub branch: Option<String>,
    /// 远程 URL
    pub remote_url: Option<String>,
    /// 最后提交 Hash
    pub last_commit: Option<String>,
    /// 是否有未提交更改
    pub has_changes: bool,
    /// 未追踪文件数
    pub untracked_count: usize,
}

pub fn get_git_info(path: &Path) -> Result<GitInfo, WorkspaceError> {
    let repo = git2::Repository::discover(path)?;

    let branch = repo.head()
        .ok()
        .and_then(|h| h.shorthand().map(String::from));

    let remote_url = repo.find_remote("origin")
        .ok()
        .and_then(|r| r.url().map(String::from));

    let last_commit = repo.head()
        .ok()
        .and_then(|h| h.peel_to_commit().ok())
        .map(|c| c.id().to_string()[..8].to_string());

    let statuses = repo.statuses(None)?;
    let has_changes = !statuses.is_empty();
    let untracked_count = statuses.iter()
        .filter(|s| s.status().contains(git2::Status::WT_NEW))
        .count();

    Ok(GitInfo {
        branch,
        remote_url,
        last_commit,
        has_changes,
        untracked_count,
    })
}
```

---

## 七、使用示例

```rust
use sage_core::workspace::{WorkspaceAnalyzer, WorkspaceConfig};

// 创建分析器
let config = WorkspaceConfig::default();
let analyzer = WorkspaceAnalyzer::new(config);

// 分析工作区
let result = analyzer.analyze(Path::new(".")).await?;

// 输出项目类型
println!("Primary language: {:?}", result.project_type.primary_language);
println!("Frameworks: {:?}", result.project_type.frameworks);

// 输出文件统计
println!("Total files: {}", result.file_stats.total_files);
println!("By extension: {:?}", result.file_stats.by_extension);

// 输出重要文件
for file in &result.important_files {
    println!("{}: {:?}", file.path.display(), file.file_type);
}

// 输出 Git 信息
if let Some(git) = &result.git_info {
    println!("Branch: {:?}", git.branch);
    println!("Has changes: {}", git.has_changes);
}
```

---

## 八、开发指南

### 8.1 添加新语言检测器

1. 创建检测器：
```rust
// workspace/detector/detectors/new_lang.rs
pub struct NewLangDetector;

impl LanguageDetector for NewLangDetector {
    fn detect(&self, path: &Path) -> Result<Option<DetectionResult>, WorkspaceError> {
        // 实现检测逻辑
    }

    fn language(&self) -> LanguageType {
        LanguageType::NewLang
    }
}
```

2. 在 `LanguageType` 添加变体

3. 添加语言模式到 `language_patterns.rs`

4. 在 `ProjectTypeDetector::new()` 注册检测器

### 8.2 添加依赖分析器

在 `dependencies/` 目录添加新分析器，实现解析逻辑。

---

## 九、相关模块

- `sage-tool-development` - 工具开发（工作区上下文）
- `sage-agent-execution` - Agent 执行（工作区信息）
- `sage-prompts` - Prompt 生成（项目上下文注入）

---

*最后更新: 2026-01-10*
