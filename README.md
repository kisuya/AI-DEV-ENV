# Terminal IDE: Ghostty + tmux + Neovim + yazi

> AI 시대의 멀티 컨텍스트 개발 환경.
> Claude Code, Copilot, aider 같은 AI 도구를 여러 프로젝트에 걸쳐 동시에 운용하기 위한 터미널 기반 워크스테이션.
> macOS 기준 작성.

```
┌─────────────────────────────────────────────────────────────────┐
│ Ghostty + tmux                                                  │
│ ┌──────────────────────────┐  ┌──────────────────────────────┐  │
│ │ Window 1: code           │  │ Window 2: server             │  │
│ │                          │  │ ┌────────────┬─────────────┐ │  │
│ │   Neovim (LazyVim)       │  │ │ 서버 실행   │ 로그 모니터링│ │  │
│ │   + 파일 트리             │  │ └────────────┴─────────────┘ │  │
│ │   + LSP 자동완성          │  ├──────────────────────────────┤  │
│ │   + 퍼지 검색             │  │ Window 3: AI + terminal      │  │
│ │                          │  │ ┌──────────────────────────┐ │  │
│ │  ┌─────────────────────┐ │  │ │ Claude Code · git · 빌드 │ │  │
│ │  │ yazi / lazygit 팝업  │ │  │ └──────────────────────────┘ │  │
│ │  └─────────────────────┘ │  └──────────────────────────────┘  │
│ └──────────────────────────┘                                    │
│ [1:code]  [2:server]  [3:ai]               main  15:42          │
│ ◀━━━━━━━━━━━━━━━━━━ 상태바 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━▶  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 목차

1. [왜 이 조합인가](#왜-이-조합인가)
2. [1단계: 전체 설치](#1단계-전체-설치)
3. [2단계: Ghostty — 터미널 에뮬레이터](#2단계-ghostty--터미널-에뮬레이터)
4. [3단계: 셸 환경 설정](#3단계-셸-환경-설정)
5. [4단계: tmux 설정](#4단계-tmux-설정)
6. [5단계: Neovim 설정 (LazyVim)](#5단계-neovim-설정-lazyvim)
7. [6단계: yazi 설정 (선택)](#6단계-yazi-설정-선택)
8. [7단계: 프로젝트 세션 자동화](#7단계-프로젝트-세션-자동화)
9. [AI 워크플로우](#ai-워크플로우)
10. [단축키 치트시트](#단축키-치트시트)
11. [학습 가이드](#학습-가이드)
12. [트러블슈팅](#트러블슈팅)
13. [다음 단계](#다음-단계)
14. [참고 자료](#참고-자료)

---

## 왜 이 조합인가

### 문제: AI 시대의 컨텍스트 스위칭

AI 코딩 도구를 실무에 도입하면 작업 패턴이 바뀐다.

- Claude Code 세션이 한 프로젝트에서 코드를 생성하는 동안, 다른 프로젝트의 서버 로그를 확인해야 한다.
- AI가 제안한 코드를 리뷰하면서, 동시에 다른 컨텍스트에서 테스트를 돌려야 한다.
- 3~5개 프로젝트를 오가며 각각의 AI 세션을 유지해야 한다.

GUI IDE는 이 워크플로우에 구조적 한계가 있다. 에디터를 닫으면 터미널 프로세스가 함께 종료되고, 프로젝트마다 무거운 인스턴스를 별도로 띄워야 한다.

### 해결: 세션 기반 터미널 워크스테이션

이 조합은 **세션이 죽지 않는 환경**을 만든다.

| 특성 | GUI IDE (VS Code 등) | Ghostty + tmux + Neovim + yazi |
|---|---|---|
| **AI 세션 영속성** | 에디터 종료 시 터미널 프로세스도 종료 | detach/attach로 AI 세션이 영구 유지 |
| **멀티 프로젝트** | 프로젝트마다 별도 윈도우 (~500MB+) | 터미널 탭 `Cmd+1/2/3`으로 즉시 전환 |
| **원격 작업** | Remote SSH 확장 필요 | SSH 접속 후 동일한 환경 |
| **리소스** | 프로젝트 5개 = 2.5GB+ | 프로젝트 5개 = 경량 |
| **에디터 기능** | 기본 제공 (풍부함) | LazyVim으로 IDE급 환경 구축 |

> VS Code는 에디터 자체로는 우수하다. 이 조합이 대체하려는 것은 에디터가 아니라, **여러 컨텍스트를 오가며 AI 도구를 동시에 운용하는 워크스테이션 레이어**다.

### 각 도구의 역할

```
Ghostty ─── GPU 가속 터미널. 이미지 미리보기, true color를 네이티브 지원한다.
tmux    ─── 세션 관리. 프로젝트별 작업 공간을 분리하고, AI 세션을 영속시킨다.
neovim  ─── 코드 편집. LSP 자동완성, 검색, 포맷팅을 제공한다.
yazi    ─── 파일 탐색. 프로젝트 구조를 시각적으로 파악하고 파일을 관리한다.
```

---

## 1단계: 전체 설치

### 핵심 도구

```bash
brew install ghostty tmux neovim
```

| 도구 | 역할 |
|---|---|
| **[Ghostty](https://ghostty.org/)** | GPU 가속 네이티브 터미널 에뮬레이터. 빠른 렌더링, 이미지 프로토콜 지원. |
| **[tmux](https://github.com/tmux/tmux)** | 터미널 멀티플렉서. 세션 영속성, 윈도우/패널 관리. |
| **[neovim](https://github.com/neovim/neovim)** | vim의 현대적 포크. LSP, 플러그인, 비동기 처리 기본 지원. |

### 선택: TUI 파일 매니저

> yazi는 터미널 기반 파일 매니저다. 파일 복사/이동/삭제, 이미지 미리보기 등 파일 관리 작업을 터미널에서 하고 싶을 때 유용하다. 코딩 중 파일 탐색만 필요하다면 nvim 내장 파일 트리(neo-tree)로 충분하므로 **설치하지 않아도 된다.**

```bash
brew install yazi
```

| 도구 | 역할 |
|---|---|
| **[yazi](https://github.com/sxyazi/yazi)** | Rust 기반 TUI 파일 매니저. 코드 하이라이팅, 이미지 미리보기 지원. |

### 보조 CLI 도구

```bash
brew install fzf fd ripgrep bat eza zoxide lazygit starship
```

| 도구 | 역할 | 대체 대상 |
|---|---|---|
| **[fzf](https://github.com/junegunn/fzf)** | 퍼지 파인더. 파일, 히스토리, 브랜치를 실시간 필터링 | `find` + `grep` |
| **[fd](https://github.com/sharkdp/fd)** | 빠른 파일 검색. fzf, yazi의 내부 엔진으로 사용 | `find` |
| **[ripgrep](https://github.com/BurntSushi/ripgrep)** | 파일 내용 검색. `.gitignore` 자동 인식 | `grep -r` |
| **[bat](https://github.com/sharkdp/bat)** | 문법 하이라이팅이 있는 파일 뷰어 | `cat` |
| **[eza](https://github.com/eza-community/eza)** | 아이콘, 색상, Git 상태가 포함된 파일 목록 | `ls`, `tree` |
| **[zoxide](https://github.com/ajeetdsouza/zoxide)** | 방문 빈도를 학습하는 스마트 디렉토리 이동 | `cd` |
| **[lazygit](https://github.com/jesseduffield/lazygit)** | TUI Git 클라이언트. 커밋, 브랜치, diff를 시각적으로 수행 | `git` CLI |
| **[starship](https://starship.rs/)** | 크로스셸 프롬프트. git 브랜치, 언어 버전 등 컨텍스트 표시 | 기본 프롬프트 |

### 선택: yazi 미리보기 의존성

```bash
brew install ffmpegthumbnailer poppler imagemagick
```

| 도구 | 목적 |
|---|---|
| **ffmpegthumbnailer** | 비디오 파일 썸네일 미리보기 |
| **poppler** | PDF 파일 미리보기 |
| **imagemagick** | 이미지 변환/미리보기 |

### Nerd Font

```bash
brew install --cask font-jetbrains-mono-nerd-font
```

Nerd Font는 개발용 폰트에 아이콘 글리프를 합친 폰트다. yazi, eza, neovim 파일 트리 등에서 파일 종류별 아이콘을 표시하는 데 필요하다. 없으면 아이콘이 깨진 문자(□, ?)로 표시된다.

### tmux 플러그인 매니저 + LazyVim

```bash
# TPM (tmux Plugin Manager)
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# LazyVim starter
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
```

---

## 2단계: Ghostty — 터미널 에뮬레이터

GPU 가속 네이티브 터미널. 모든 TUI 도구의 기반.

**왜 Ghostty인가:** 빠른 렌더링, yazi 이미지 미리보기 지원(Kitty 이미지 프로토콜), true color, tmux에 최적화된 최소 설정.

### 설정 파일: `~/.config/ghostty/config`

```
# ── 폰트 ──
font-family = "JetBrains Mono"
font-size = 14

# ── 테마 (전체 도구와 통일) ──
theme = Catppuccin Mocha

# ── 이미지 프로토콜 (yazi 미리보기에 필요) ──
image-storage-limit = 320000000

# ── 창 설정 ──
window-padding-x = 8
window-padding-y = 8

# ── 클립보드 (OSC 52 — SSH 원격에서도 작동) ──
clipboard-read = allow
clipboard-write = allow

# ── 커서 ──
cursor-style = bar
cursor-style-blink = false

# ── 마우스 ──
mouse-scroll-multiplier = 3
```

### 설정 항목 설명

| 설정 | 목적 |
|---|---|
| `font-family` | Nerd Font 호환 코딩 폰트. 아이콘 표시에 필요 |
| `theme` | Catppuccin Mocha — 전체 도구와 테마 통일 |
| `image-storage-limit` | yazi에서 이미지 미리보기 시 메모리 할당량 (300MB+) |
| `clipboard-read/write = allow` | OSC 52 기반 클립보드. SSH 원격에서도 작동 |
| `cursor-style-blink = false` | 깜빡임 제거. 시각적 노이즈 감소 |

---

## 3단계: 셸 환경 설정

이후 단계에서 yazi, lazygit 등이 neovim을 기본 에디터로 인식하도록 환경 변수와 셸 도구를 설정한다.

### `~/.zshrc` 에 추가

```bash
# ── 기본 에디터 ──
export EDITOR="nvim"
export VISUAL="nvim"

# ── dev 스크립트 경로 ──
export PATH="$HOME/.local/bin:$PATH"

# ── zoxide — cd 대체, 자주 가는 디렉토리 학습 ──
eval "$(zoxide init zsh)"
# 사용: z blog → ~/Dev/blog 으로 점프

# ── fzf — fuzzy finder 셸 통합 ──
source <(fzf --zsh)
# Ctrl+R: 명령어 히스토리 검색
# Ctrl+T: 파일 검색
# Alt+C: 디렉토리 이동

# ── starship — 프롬프트 (선택 사항) ──
# 기존 oh-my-zsh 테마를 사용하는 경우 충돌하므로 아래 줄을 주석 처리한다.
# 사용하려면 ZSH_THEME=""로 변경한 후 주석을 해제한다.
# eval "$(starship init zsh)"

# ── bat — cat 대체 (구문 강조) ──
alias cat="bat"

# ── eza — ls 대체 (아이콘, git 상태 표시) ──
alias ls="eza --icons --group-directories-first"
alias ll="eza --icons --group-directories-first -la"
alias tree="eza --icons --tree"

# ── lazygit ──
alias lg="lazygit"

# ── yazi 셸 통합 (종료 시 마지막 디렉토리로 이동) ──
function y() {
  local tmp="$(mktemp -t "yazi-cwd.XXXXXX")" cwd
  yazi "$@" --cwd-file="$tmp"
  if cwd="$(command cat -- "$tmp")" && [ -n "$cwd" ] && [ "$cwd" != "$PWD" ]; then
    builtin cd -- "$cwd"
  fi
  rm -f -- "$tmp"
}
```

> `yazi` 대신 **`y`**를 입력하면 종료 시 마지막 탐색 디렉토리로 셸이 이동한다.

### fzf 단축키

| 단축키 | 동작 |
|---|---|
| `Ctrl+R` | 명령어 히스토리 fuzzy 검색 |
| `Ctrl+T` | 현재 디렉토리 아래 파일 fuzzy 검색 |
| `Alt+C` | 디렉토리 fuzzy 검색 후 이동 |

---

## 4단계: tmux 설정

### TPM 설치

[TPM (tmux Plugin Manager)](https://github.com/tmux-plugins/tpm)은 tmux 플러그인을 선언만 하면 자동으로 설치/업데이트를 관리한다.

```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

### 설정 파일: `~/.tmux.conf`

아래 설정은 7개 블록으로 구성되어 있다. 각 블록은 특정 불편을 해결한다.

```bash
# ══════════════════════════════════════════════
# 블록 1: 기본 옵션
# ══════════════════════════════════════════════

# prefix를 Ctrl+a로 변경
# 기본값 Ctrl+b는 키보드 중앙이라 한 손으로 누르기 어렵다.
# Ctrl+a는 왼손 새끼+검지로 빠르게 누를 수 있다.
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# 설정 리로드 단축키
bind r source-file ~/.tmux.conf \; display "Reloaded!"

# True Color + 256색 지원
# 없으면 neovim, bat 등의 색상 테마가 깨진다.
set -g default-terminal "tmux-256color"
set -ag terminal-overrides ",xterm-256color:RGB"

# 마우스 지원
set -g mouse on

# ESC 딜레이 제거
# 기본값 500ms. neovim에서 ESC 반응이 0.5초 지연되는 것을 방지한다.
set -sg escape-time 0

# 포커스 이벤트 전달
# neovim이 패널 포커스 변경을 감지하여 파일 자동 갱신(autoread)을 할 수 있게 한다.
set -g focus-events on

# 히스토리 버퍼 확장 (기본 2000줄 → 50000줄)
# AI 도구의 긴 출력이나 서버 로그를 스크롤해서 볼 때 필요하다.
set -g history-limit 50000

# 인덱스 1부터 시작
# Cmd+1/2/3과 직관적으로 대응시키기 위함.
set -g base-index 1
setw -g pane-base-index 1

# 윈도우 번호 자동 재정렬
set -g renumber-windows on

# 클립보드 연동 (OSC 52)
set -g set-clipboard on

# yazi 이미지 미리보기를 위한 패스스루 허용
set -g allow-passthrough all
set -ga update-environment TERM
set -ga update-environment TERM_PROGRAM


# ══════════════════════════════════════════════
# 블록 2: 패널(Pane) 관리
# ══════════════════════════════════════════════

# 직관적 분할 키바인딩 (|: 수직, -: 수평)
# 새 패널은 현재 디렉토리에서 열린다.
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
unbind '"'
unbind %

# 새 윈도우도 현재 경로에서
bind c new-window -c "#{pane_current_path}"

# vim 스타일 패널 이동 (prefix + h/j/k/l)
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# Alt+방향키로 빠른 이동 (prefix 불필요)
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

# 패널 리사이즈 (대문자 H/J/K/L, 5칸씩)
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# 패널 최대화 토글
bind m resize-pane -Z

# 윈도우 전환 (Shift+좌/우, prefix 불필요)
bind -n S-Left previous-window
bind -n S-Right next-window

# 세션 전환 단축키
bind s choose-tree -sZ


# ══════════════════════════════════════════════
# 블록 3: 복사 모드
# ══════════════════════════════════════════════

# vi 스타일 복사 (neovim과 동일한 키바인딩)
setw -g mode-keys vi
bind -T copy-mode-vi v send -X begin-selection
bind -T copy-mode-vi y send -X copy-pipe-and-cancel "pbcopy"


# ══════════════════════════════════════════════
# 블록 4: 팝업 바인딩 (tmux 3.2+)
# ══════════════════════════════════════════════
# display-popup으로 현재 레이아웃 위에 떠 있는 창을 띄운다.
# 팝업은 열었다 닫아도 기존 레이아웃이 유지된다.

# prefix+e: yazi 파일 매니저 팝업
# tmux popup은 passthrough를 지원하지 않아 yazi가 터미널을 감지하지 못한다.
# 워크어라운드: 팝업 안에서 새 tmux 세션을 만들어 실행한다.
# ref: https://github.com/sxyazi/yazi/issues/2308
bind e display-popup -d "#{pane_current_path}" -w 90% -h 85% -E 'tmux new-session yazi \; set status off'

# prefix+g: lazygit 팝업
bind g display-popup -d "#{pane_current_path}" -w 90% -h 90% -E "lazygit"

# prefix+f: fzf로 파일 찾아서 neovim으로 열기
bind f display-popup -d "#{pane_current_path}" -w 80% -h 80% -E \
    'file=$(fzf --preview "bat --color=always --style=numbers {}") && nvim "$file"'

# prefix+t: 임시 터미널
bind t display-popup -d "#{pane_current_path}" -w 80% -h 80% -E "zsh"


# ══════════════════════════════════════════════
# 블록 5: 상태바
# ══════════════════════════════════════════════
# 세션명, 현재 윈도우, git 브랜치, PREFIX 표시, 시간을 표시하여
# 어느 프로젝트의 어느 윈도우에 있는지 한눈에 파악할 수 있다.

set -g status-position top
set -g status-interval 5
set -g status-style "bg=#1e1e2e,fg=#cdd6f4"
set -g status-left "#[fg=#1e1e2e,bg=#89b4fa,bold] #S "
set -g status-left-length 30
setw -g window-status-format " #[fg=#6c7086]#I:#W "
setw -g window-status-current-format "#[fg=#1e1e2e,bg=#a6e3a1,bold] #I:#W "
set -g status-right "#{?client_prefix,#[bg=#f38ba8,fg=#1e1e2e,bold] PREFIX ,}#[fg=#a6e3a1] #(cd #{pane_current_path}; git branch --show-current 2>/dev/null) #[fg=#f9e2af]%H:%M "
set -g status-right-length 60
set -g pane-border-style "fg=#313244"
set -g pane-active-border-style "fg=#89b4fa"


# ══════════════════════════════════════════════
# 블록 6: 플러그인
# ══════════════════════════════════════════════

set -g @plugin 'tmux-plugins/tpm'

# tmux-resurrect: 세션 상태를 디스크에 저장.
# 시스템 재시작 후에도 윈도우/패널 배치를 복원할 수 있다.
set -g @plugin 'tmux-plugins/tmux-resurrect'

# tmux-continuum: 15분마다 자동 저장 + 시작 시 자동 복원.
set -g @plugin 'tmux-plugins/tmux-continuum'

# tmux-yank: 복사한 텍스트를 시스템 클립보드에 연결.
set -g @plugin 'tmux-plugins/tmux-yank'

# vim-tmux-navigator: Ctrl+h/j/k/l로 neovim과 tmux 패널을 구분 없이 이동.
set -g @plugin 'christoomey/vim-tmux-navigator'

# resurrect: nvim 세션도 함께 복원
set -g @resurrect-strategy-nvim 'session'
set -g @resurrect-capture-pane-contents 'on'

# continuum: 15분마다 자동 저장 + 시작 시 복원
set -g @continuum-restore 'on'
set -g @continuum-save-interval '15'

# TPM 초기화 (반드시 마지막 줄)
run '~/.tmux/plugins/tpm/tpm'
```

### 플러그인 설치

```bash
# 1. tmux 실행
tmux

# 2. tmux 안에서 플러그인 설치
#    Ctrl+a 를 누른 후 Shift+I (대문자 I)
#    하단에 설치 진행 메시지가 표시된다. 완료되면 Enter.
```

### 플러그인 목록

| 플러그인 | 목적 |
|---|---|
| **tmux-resurrect** | 재부팅 후에도 세션 레이아웃(윈도우, 패널 배치) 복원 |
| **tmux-continuum** | resurrect를 15분마다 자동 저장. 수동 저장 불필요 |
| **tmux-yank** | 복사 모드에서 시스템 클립보드(pbcopy)로 자동 복사 |
| **vim-tmux-navigator** | Ctrl+h/j/k/l로 tmux 패널↔nvim 윈도우를 경계 없이 이동 |

### 설정 항목 설명

| 설정 | 목적 |
|---|---|
| `prefix C-a` | Ctrl+A가 Ctrl+B보다 손에 가까움 |
| `bind r source-file` | 설정 변경 후 prefix+r로 즉시 리로드 |
| `escape-time 0` | Neovim에서 ESC 누르면 즉시 반응 (기본값은 500ms 지연) |
| `focus-events on` | Neovim이 포커스 변경을 감지해서 파일 자동 리로드 |
| `mouse on` | 마우스로 패널 선택, 리사이즈, 스크롤 가능 |
| `mode-keys vi` | 복사 모드에서 vi 키바인딩 사용 |
| `renumber-windows on` | 윈도우 3을 닫으면 4→3으로 자동 재번호 |
| `set-clipboard on` | OSC 52 클립보드 연동. SSH 원격에서도 복사 가능 |

---

## 5단계: Neovim 설정 (LazyVim)

### LazyVim이란

Neovim은 기본 상태에서는 텍스트 편집만 되는 빈 에디터다. **[LazyVim](https://www.lazyvim.org/)** 은 LSP, 파일 트리, 검색, 포맷팅 등 플러그인 조합을 미리 구성해놓은 배포판으로, 설치 즉시 IDE급 환경이 갖춰진다.

| 내장 기능 | 플러그인 | 설명 |
|---|---|---|
| 파일 트리 사이드바 | neo-tree.nvim | 왼쪽 사이드바에 폴더 구조 표시 |
| 파일 검색 (fuzzy) | telescope.nvim | Cmd+P 같은 파일 열기 |
| 텍스트 검색 (grep) | telescope + rg | 프로젝트 전체 텍스트 검색 |
| LSP (자동완성, 진단) | nvim-lspconfig | 자동완성, 에러 표시, 정의로 이동 |
| Git 상태 표시 | gitsigns.nvim | 줄 단위 git 변경사항 표시 |
| 코드 포맷팅 | conform.nvim | Prettier, Black 등 저장 시 자동 포맷 |
| 문법 강조 | treesitter | AST 기반 정확한 구문 색칠 |
| 키 가이드 | which-key.nvim | 단축키를 누르면 가능한 키 목록 표시 |
| 탭 바 | bufferline.nvim | 열린 파일을 상단 탭으로 표시 |

### LazyVim 설치

기존 neovim 설정이 있다면 먼저 백업한다:

```bash
# 기존 설정 백업 (있는 경우)
mv ~/.config/nvim ~/.config/nvim.backup 2>/dev/null
mv ~/.local/share/nvim ~/.local/share/nvim.backup 2>/dev/null
mv ~/.local/state/nvim ~/.local/state/nvim.backup 2>/dev/null
mv ~/.cache/nvim ~/.cache/nvim.backup 2>/dev/null
```

```bash
# LazyVim starter 설치
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
```

첫 실행 시 자동으로 플러그인이 설치된다 (1~2분 소요):

```bash
nvim
```

### Language Server 설정

LazyVim을 설치한 것만으로는 LSP 자동완성이 동작하지 않는다. 사용하는 언어에 맞는 Language Server를 설치해야 한다.

**방법 1: LazyExtras로 언어 지원 활성화 (권장)**

neovim을 열고 `:LazyExtras`를 실행하면, 언어별 LSP/포맷터/린터 묶음을 한 번에 활성화할 수 있다. 사용하는 언어를 찾아서 `x`로 토글한다.

**방법 2: Mason으로 개별 설치**

neovim을 열고 `:Mason`을 실행하면 Language Server를 개별적으로 검색하고 설치할 수 있다.

| 언어 | Language Server | Mason에서 검색 |
|---|---|---|
| TypeScript/JavaScript | typescript-language-server | `typescript` |
| Python | pyright | `pyright` |
| Go | gopls | `gopls` |
| Rust | rust-analyzer | `rust` |
| Lua | lua-language-server | `lua` |

### 추가 플러그인: `~/.config/nvim/lua/plugins/custom.lua`

```lua
return {
  -- 테마 통일 (Ghostty, tmux와 동일하게 catppuccin)
  {
    "catppuccin/nvim",
    name = "catppuccin",
    lazy = false,
    priority = 1000,
    opts = {
      transparent_background = true,
    },
  },

  -- tmux 패널과 nvim 분할 간 이동 통합
  -- Ctrl+h/j/k/l 로 tmux↔nvim 경계 없이 이동
  {
    "christoomey/vim-tmux-navigator",
    keys = {
      { "<C-h>", "<cmd>TmuxNavigateLeft<cr>",  desc = "Go to left pane" },
      { "<C-j>", "<cmd>TmuxNavigateDown<cr>",  desc = "Go to lower pane" },
      { "<C-k>", "<cmd>TmuxNavigateUp<cr>",    desc = "Go to upper pane" },
      { "<C-l>", "<cmd>TmuxNavigateRight<cr>", desc = "Go to right pane" },
    },
  },

  -- 마크다운 미리보기 (블로그, 문서 작업용)
  {
    "MeanderingProgrammer/render-markdown.nvim",
    ft = { "markdown", "mdx" },
  },
}
```

### 추가 플러그인 설명

| 플러그인 | 목적 |
|---|---|
| **catppuccin** | Ghostty, tmux, nvim 전체를 동일한 색상 테마로 통일. 투명 배경 지원. |
| **vim-tmux-navigator** | tmux 패널↔nvim 분할 사이를 Ctrl+h/j/k/l로 끊김 없이 이동 |
| **render-markdown.nvim** | 마크다운/MDX 파일을 nvim 안에서 렌더링된 형태로 미리보기 |

---

## 6단계: yazi 설정 (선택)

### 기본 설정: `~/.config/yazi/yazi.toml`

```bash
mkdir -p ~/.config/yazi
```

```toml
[manager]
show_hidden = true          # 숨김 파일(.env, .gitignore 등) 표시
sort_by = "natural"         # 자연어 순서 정렬 (file1 < file2 < file10)
sort_dir_first = true       # 디렉토리를 파일보다 위에 표시
linemode = "size"           # 파일 크기를 오른쪽에 표시

[preview]
max_width = 1000            # 미리보기 최대 너비
max_height = 1000           # 미리보기 최대 높이

[opener]
edit = [
    { run = 'nvim "$@"', block = true, desc = "Neovim" },
]

[open]
rules = [
    { mime = "text/*", use = "edit" },
    { name = "*.py",   use = "edit" },
    { name = "*.js",   use = "edit" },
    { name = "*.ts",   use = "edit" },
    { name = "*.tsx",  use = "edit" },
    { name = "*.jsx",  use = "edit" },
    { name = "*.json", use = "edit" },
    { name = "*.yaml", use = "edit" },
    { name = "*.yml",  use = "edit" },
    { name = "*.toml", use = "edit" },
    { name = "*.md",   use = "edit" },
    { name = "*.sh",   use = "edit" },
    { name = "*.lua",  use = "edit" },
    { name = "*.html", use = "edit" },
    { name = "*.css",  use = "edit" },
    { name = "*.env*", use = "edit" },
    { name = "Dockerfile*", use = "edit" },
    { name = "Makefile", use = "edit" },
]
```

### 키맵 설정: `~/.config/yazi/keymap.toml`

```toml
# Enter: 기본 열기 (yazi 안에서 neovim이 열림)
[[manager.prepend_keymap]]
on   = ["<Enter>"]
run  = "open"
desc = "Open file (nvim)"

# V: 옆 tmux pane의 neovim에서 열기
# 파일 트리(yazi)를 보면서 편집할 수 있다.
[[manager.prepend_keymap]]
on   = ["V"]
run  = '''
    shell 'tmux send-keys -t "{right}" "nvim \"$0\"" Enter' --confirm
'''
desc = "Open in nvim (right tmux pane)"

# 자주 쓰는 디렉토리로 빠르게 이동
[[manager.prepend_keymap]]
on   = ["g", "d"]
run  = "cd ~/Dev"
desc = "Go to Dev directory"

[[manager.prepend_keymap]]
on   = ["g", "c"]
run  = "cd ~/.config"
desc = "Go to config directory"
```

---

## 7단계: 프로젝트 세션 자동화

매번 tmux 윈도우를 수동으로 만들고 패널을 분할하는 반복 작업을 자동화한다. 두 가지 방법 중 선택한다.

### 방법 A: `dev` 스크립트 (의존성 없음, 권장)

```bash
mkdir -p ~/.local/bin

cat > ~/.local/bin/dev << 'DEVEOF'
#!/bin/bash
# ──────────────────────────────────────────────
# dev — 프로젝트별 tmux 개발 세션 자동 생성
# Usage: dev [project-path] [session-name]
# Example: dev ~/work/my-api api
# ──────────────────────────────────────────────

PROJECT_PATH="${1:-.}"
PROJECT_PATH="$(cd "$PROJECT_PATH" && pwd)"
SESSION_NAME="${2:-$(basename "$PROJECT_PATH")}"

# 세션이 이미 있으면 attach
if tmux has-session -t "$SESSION_NAME" 2>/dev/null; then
    tmux attach-session -t "$SESSION_NAME"
    exit 0
fi

cd "$PROJECT_PATH" || exit 1

# ┌─────────────────────────────────────┐
# │ Window 1: code                      │
# │  neovim (전체)                      │
# ├─────────────────────────────────────┤
# │ Window 2: server                    │
# │  좌: 서버    │  우: 로그/출력       │
# ├─────────────────────────────────────┤
# │ Window 3: ai                        │
# │  상: AI 도구 (Claude Code 등)       │
# │  하: git/빌드/범용                  │
# └─────────────────────────────────────┘

tmux new-session -d -s "$SESSION_NAME" -n "code" -c "$PROJECT_PATH"
tmux send-keys -t "$SESSION_NAME:code" "nvim ." C-m

tmux new-window -t "$SESSION_NAME" -n "server" -c "$PROJECT_PATH"
tmux split-window -h -t "$SESSION_NAME:server" -c "$PROJECT_PATH"

tmux new-window -t "$SESSION_NAME" -n "ai" -c "$PROJECT_PATH"
tmux split-window -v -t "$SESSION_NAME:ai" -c "$PROJECT_PATH"

tmux select-window -t "$SESSION_NAME:code"
tmux attach-session -t "$SESSION_NAME"
DEVEOF

chmod +x ~/.local/bin/dev
```

### 방법 B: tmuxinator (YAML 기반, 여러 프로젝트를 선언적으로 관리)

프로젝트마다 다른 레이아웃이 필요하거나, 여러 프로젝트 세션을 하나의 명령으로 동시에 띄우고 싶을 때 적합하다.

```bash
brew install tmuxinator
```

#### 설정 파일 예시: `~/.tmuxinator/dev.yml`

```yaml
name: dev
windows:
  - blog:
      root: ~/Dev/blog
      panes:
        - claude
  - blog-feature:
      root: ~/Dev/blog-worktree/feature-x
      panes:
        - claude
  - api:
      root: ~/Dev/api
      layout: main-horizontal
      panes:
        - claude
        - # 빈 셸 (명령어용)
```

#### 사용법

```bash
tmuxinator start dev   # 전체 환경 한 번에 실행
tmuxinator stop dev    # 전체 종료
tmuxinator edit dev    # 설정 편집
tmuxinator list        # 저장된 워크스페이스 목록
```

---

## AI 워크플로우

이 환경의 핵심 가치는 **AI 세션의 영속성과 멀티 컨텍스트 전환**이다.

### 기본 워크플로우

```
1. dev 명령으로 프로젝트 세션 생성
   → dev ~/work/my-project

2. Window 1 (code): neovim에서 코딩
   → Space+e 파일 트리, Space+ff 파일 검색, gd 정의 이동

3. Window 2 (server): 서버 실행
   → Shift+→ 로 이동, npm run dev 등 실행

4. Window 3 (ai): AI 도구 실행
   → Shift+→ 로 이동, claude 등 AI CLI 도구 실행
   → 하단 패널에서 git, 빌드, 테스트

5. prefix+e: yazi 팝업으로 파일 탐색 (레이아웃 유지)
6. prefix+g: lazygit 팝업으로 Git 작업 (레이아웃 유지)
7. prefix+f: fzf 팝업으로 파일 찾아서 nvim으로 열기

8. 퇴근: prefix+d 로 detach
   → AI 세션, 서버 프로세스 모두 백그라운드에 유지

9. 다음 날: tmux a -t my-project
   → 어제 상태 그대로 복원
```

### 멀티 프로젝트 전환

각 터미널 탭에 독립적인 tmux 세션을 할당하면 `Cmd+1/2/3`으로 프로젝트 간 즉시 전환할 수 있다.

```bash
# 터미널 탭 1 (Cmd+1): 프론트엔드 — Claude Code로 UI 생성 중
dev ~/work/frontend fe

# 터미널 탭 2 (Cmd+2): 백엔드 API — Claude Code로 엔드포인트 작업 중
dev ~/work/backend api

# 터미널 탭 3 (Cmd+3): 인프라 — 배포 스크립트 작업
dev ~/work/infra infra
```

핵심은 **프로젝트 A에서 AI가 코드를 생성하는 동안 프로젝트 B로 전환하여 리뷰/테스트를 병행**할 수 있다는 점이다. `Cmd+1`로 돌아오면 AI 세션은 그대로 살아있다.

### AI 도구별 활용

| 도구 | 이 환경에서의 활용 |
|---|---|
| **Claude Code** | tmux 패널에서 실행. detach 후에도 세션 유지. 여러 프로젝트에서 동시 운용 가능. |
| **GitHub Copilot** | neovim 플러그인으로 설치. LazyVim의 Extras에서 `coding.copilot` 활성화. |
| **aider** | tmux 패널에서 실행. Claude Code와 동일하게 세션 영속성 확보. |

### 테마 통일

모든 도구가 **Catppuccin Mocha** 테마를 사용하도록 설정되어 시각적 일관성이 유지된다.

| 도구 | 설정 |
|---|---|
| Ghostty | `theme = Catppuccin Mocha` |
| tmux | 상태바 색상 `#1e1e2e`, `#cdd6f4` (catppuccin 팔레트) |
| Neovim | catppuccin 플러그인 (transparent_background) |
| bat | `bat --theme="Catppuccin Mocha"` 또는 `~/.config/bat/config`에 설정 |
| lazygit | catppuccin 테마 적용 가능 |

---

## 단축키 치트시트

### tmux (prefix = `Ctrl+a`)

| 카테고리 | 동작 | 단축키 |
|---|---|---|
| **세션** | detach (나가기, 세션 유지) | `prefix + d` |
| | 세션 목록/전환 | `prefix + s` |
| | 설정 리로드 | `prefix + r` |
| **윈도우** | 새 윈도우 | `prefix + c` |
| | 다음/이전 윈도우 | `Shift + →/←` |
| | 번호로 이동 | `prefix + 1~9` |
| | 이름 변경 | `prefix + ,` |
| **패널** | 수직 분할 | `prefix + \|` |
| | 수평 분할 | `prefix + -` |
| | 패널 이동 (vim) | `prefix + h/j/k/l` |
| | 패널 이동 (빠른) | `Alt + 방향키` |
| | nvim↔tmux 통합 이동 | `Ctrl + h/j/k/l` |
| | 최대화 토글 | `prefix + m` |
| | 리사이즈 | `prefix + H/J/K/L` |
| | 닫기 | `prefix + x` |
| **팝업** | yazi (파일 매니저) | `prefix + e` |
| | lazygit (Git) | `prefix + g` |
| | fzf 파일 검색 → nvim | `prefix + f` |
| | 임시 터미널 | `prefix + t` |
| **복사** | 복사 모드 진입 | `prefix + [` |
| | 선택 시작 (복사 모드) | `v` |
| | 복사 (복사 모드) | `y` |
| **세션 관리** | 세션 저장 (resurrect) | `prefix + Ctrl+s` |
| | 세션 복원 (resurrect) | `prefix + Ctrl+r` |
| **플러그인** | TPM 플러그인 설치 | `prefix + I` |

### Neovim / LazyVim

| 카테고리 | 동작 | 단축키 |
|---|---|---|
| **기본** | 명령 가이드 (which-key) | `Space` |
| | 저장 | `:w` |
| | 종료 | `:q` |
| | 편집 모드 진입 | `i` |
| | 노멀 모드 복귀 | `Esc` |
| **파일** | 파일 트리 토글 | `Space + e` |
| | 파일 검색 | `Space + f + f` |
| | 최근 파일 | `Space + f + r` |
| | 텍스트 검색 (grep) | `Space + /` |
| **이동** | 정의로 이동 | `gd` |
| | 참조 검색 | `gr` |
| | 문서/타입 정보 | `K` |
| | 이전 위치로 돌아가기 | `Ctrl + o` |
| **편집** | 한 줄 삭제 | `dd` |
| | 한 줄 복사 | `yy` |
| | 붙여넣기 | `p` |
| | 실행 취소 / 다시 실행 | `u` / `Ctrl + r` |
| **윈도우** | tmux 패널 통합 이동 | `Ctrl + h/j/k/l` |
| **버퍼** | 열린 버퍼 목록 | `Space + ,` |
| | 현재 버퍼 닫기 | `Space + b + d` |
| **코드** | 코드 포맷팅 | `Space + c + f` |
| | 에러 목록 | `Space + x + x` |
| | 터미널 | `Ctrl + /` |

### yazi

| 카테고리 | 동작 | 단축키 |
|---|---|---|
| **탐색** | 디렉토리 이동 | `h/l` 또는 `←/→` |
| | 파일 이동 | `j/k` 또는 `↑/↓` |
| | 파일 열기 (nvim) | `Enter` |
| | 옆 tmux pane에서 열기 | `V` |
| | 종료 | `q` |
| **파일 작업** | 새 파일 / 새 디렉토리 | `a` / `A` |
| | 이름 변경 | `r` |
| | 휴지통 삭제 / 영구 삭제 | `d` / `D` |
| | 복사 / 잘라내기 / 붙여넣기 | `y` / `x` / `p` |
| | 숨김 파일 토글 | `.` |
| **검색** | 파일명 검색 | `/` |
| | zoxide 점프 | `z` |
| | 셸 명령 실행 | `s` |
| **선택** | 선택 토글 | `Space` |
| | 비주얼 모드 (다중 선택) | `v` |
| **탭** | 새 탭 / 탭 전환 / 닫기 | `t` / `1~9` / `Ctrl+c` |
| **바로가기** | ~/Dev로 이동 | `g + d` |
| | ~/.config로 이동 | `g + c` |

### 셸 (zsh)

| 도구 | 동작 | 사용법 |
|---|---|---|
| **zoxide** | 디렉토리 점프 | `z blog` → ~/Dev/blog |
| **fzf** | 히스토리 검색 | `Ctrl+R` |
| **fzf** | 파일 검색 | `Ctrl+T` |
| **fzf** | 디렉토리 이동 | `Alt+C` |
| **bat** | 파일 출력 (cat 대체) | `cat file.ts` |
| **eza** | 파일 목록 (ls 대체) | `ls`, `ll`, `tree` |
| **lazygit** | TUI Git | `lg` |
| **yazi** | 파일 매니저 | `y` |

---

## 학습 가이드

설치와 설정은 빠르게 끝나지만, 실제로 손에 익기까지는 시간이 필요하다.

### 현실적인 타임라인

| 시기 | 상태 |
|---|---|
| **1일차** | 설치 완료. tmux 윈도우 전환과 neovim 기본 편집(`i`, `Esc`, `:w`, `:q`)에 집중. |
| **1주차** | tmux prefix 키와 패널 분할이 손에 익기 시작. neovim에서 `Space` 기반 단축키를 which-key 가이드에 의존하며 사용. |
| **2주차** | 멀티 프로젝트 전환이 자연스러워짐. yazi/lazygit 팝업이 워크플로우에 통합. |
| **1개월** | 키보드에서 손이 거의 떠나지 않는 상태. GUI IDE로 돌아가고 싶지 않아짐. |

### 권장 학습 순서

1. **tmux 먼저**: 윈도우 전환(`Shift+←/→`), detach(`prefix+d`), attach(`tmux a`)만 익힌다.
2. **neovim 기본**: `i`(편집), `Esc`(노멀), `:w`(저장), `:q`(종료), `Space`(가이드)만 익힌다.
3. **팝업 활용**: `prefix+e`(yazi), `prefix+g`(lazygit)를 습관화한다.
4. **고급 기능**: LSP(`gd`, `gr`), telescope(`Space+ff`), 텍스트 검색(`Space+/`)을 점진적으로 추가한다.

> 처음부터 모든 단축키를 외우려 하지 않아도 된다. which-key가 항상 다음 키를 알려준다.

---

## 트러블슈팅

<details>
<summary><b>프롬프트가 깨진다 (starship + oh-my-zsh 충돌)</b></summary>

starship과 oh-my-zsh 테마(`ZSH_THEME="agnoster"` 등)가 동시에 활성화되면 프롬프트가 충돌하여 깨진다. 둘 중 하나만 사용해야 한다.

**기존 oh-my-zsh 테마를 유지하려면** (권장):
```bash
# ~/.zshrc 에서 starship 초기화 줄을 제거 또는 주석 처리
# eval "$(starship init zsh)"
```

**starship을 사용하려면**:
```bash
# ~/.zshrc 에서 ZSH_THEME를 비활성화
ZSH_THEME=""
# 그리고 starship 초기화 줄 주석 해제
eval "$(starship init zsh)"
```

기본적으로 기존 셸 프롬프트를 유지하는 것을 권장한다. starship은 선택 사항이다.
</details>

<details>
<summary><b>아이콘이 깨진다 (□, ? 표시)</b></summary>

Nerd Font가 설치되어 있고 Ghostty에서 해당 폰트가 선택되어 있는지 확인한다.

```bash
brew list --cask | grep nerd
```

`~/.config/ghostty/config`에서 `font-family = "JetBrains Mono"` (Nerd Font 버전)가 설정되어 있는지 확인한다.
</details>

<details>
<summary><b>색상이 이상하다</b></summary>

```bash
# tmux 안에서 확인 → tmux-256color 이어야 함
echo $TERM

# Ghostty에서 직접 확인 → xterm-ghostty 또는 xterm-256color
echo $TERM
```

`~/.tmux.conf`에 아래가 있는지 확인:
```bash
set -g default-terminal "tmux-256color"
set -ag terminal-overrides ",xterm-256color:RGB"
```
</details>

<details>
<summary><b>Neovim에서 ESC 반응이 느리다</b></summary>

`~/.tmux.conf`에 `set -sg escape-time 0`이 있는지 확인한다. 기본값이 500ms라서 ESC를 누르면 0.5초 딜레이가 생긴다.

```bash
tmux show -g escape-time
# 0 이어야 함
```
</details>

<details>
<summary><b>Ctrl+h가 동작하지 않는다 (vim-tmux-navigator)</b></summary>

일부 터미널에서 `Ctrl+h`가 `Backspace`로 매핑되어 있을 수 있다.
- **iTerm2**: Settings > Profiles > Keys > Key Mappings에서 해당 매핑을 삭제
- **Ghostty**: 기본적으로 정상 동작
</details>

<details>
<summary><b>tmux 팝업에서 yazi가 "Terminal response timeout" 에러를 표시한다</b></summary>

tmux popup은 passthrough 이스케이프 시퀀스를 지원하지 않는다 ([tmux#4329](https://github.com/tmux/tmux/issues/4329)). yazi v25.2.7+는 passthrough로 터미널을 감지하므로 팝업에서 실패한다.

**워크어라운드:** 팝업 안에서 새 tmux 세션을 만들어 실행한다 ([yazi#2308](https://github.com/sxyazi/yazi/issues/2308)):

```bash
# ~/.tmux.conf
bind e display-popup -d "#{pane_current_path}" -w 90% -h 85% -E 'tmux new-session yazi \; set status off'
```

참고: 일반 tmux pane에서는 이 문제가 발생하지 않는다.
</details>

<details>
<summary><b>yazi에서 이미지 미리보기가 안 된다</b></summary>

Ghostty는 Kitty 이미지 프로토콜을 지원하므로 기본 동작한다. macOS 기본 Terminal.app은 이미지 미리보기를 지원하지 않는다.

`~/.config/ghostty/config`에 `image-storage-limit = 320000000`이 설정되어 있는지 확인한다.

```bash
yazi --version
brew upgrade yazi
```
</details>

<details>
<summary><b>tmux 플러그인이 설치되지 않는다</b></summary>

```bash
# TPM 설치 확인
ls ~/.tmux/plugins/tpm

# 수동 설치
~/.tmux/plugins/tpm/bin/install_plugins
```
</details>

<details>
<summary><b>prefix+e 팝업이 안 뜬다</b></summary>

tmux 3.2 이상이 필요하다.

```bash
tmux -V
brew upgrade tmux
```
</details>

<details>
<summary><b>SSH 원격에서 클립보드 복사가 안 된다</b></summary>

SSH 환경에서는 `pbcopy`가 동작하지 않는다. OSC 52 이스케이프 시퀀스를 사용하면 원격에서도 로컬 클립보드로 복사할 수 있다.

tmux에서 OSC 52를 활성화하려면 `~/.tmux.conf`에 추가:
```bash
set -g set-clipboard on
```

Ghostty는 OSC 52를 기본 지원한다 (`clipboard-read/write = allow`).

neovim에서 OSC 52를 사용하려면 `~/.config/nvim/lua/plugins/osc52.lua`:
```lua
return {
  {
    "ojroques/nvim-osc52",
    config = function()
      require("osc52").setup({ trim = false })
      vim.keymap.set("n", "<leader>y", require("osc52").copy_operator, { expr = true })
      vim.keymap.set("v", "<leader>y", require("osc52").copy_visual)
    end,
  },
}
```
</details>

<details>
<summary><b>Alt+방향키 패널 이동이 안 된다</b></summary>

- **iTerm2**: Settings > Profiles > Keys > General > Left/Right Option key > `Esc+`로 변경
- **Ghostty**: 기본적으로 정상 동작
</details>

---

## 다음 단계

위 설정만으로 충분히 사용할 수 있다. 익숙해지면 하나씩 추가한다.

| 단계 | 내용 | 참고 |
|---|---|---|
| **언어별 LSP** | `:LazyExtras`에서 언어 지원 활성화 | [LazyVim Extras](https://www.lazyvim.org/extras) |
| **Copilot 연동** | `:LazyExtras`에서 `coding.copilot` 활성화 | [LazyVim Copilot](https://www.lazyvim.org/extras/coding/copilot) |
| **Neovim 플러그인** | `~/.config/nvim/lua/plugins/`에 `.lua` 파일 추가 | [LazyVim Docs](https://www.lazyvim.org/) |
| **yazi 테마** | Catppuccin 등 테마 설치 | [yazi-rs/flavors](https://github.com/yazi-rs/flavors) |
| **bat 테마** | `~/.config/bat/config`에 `--theme="Catppuccin Mocha"` | [bat docs](https://github.com/sharkdp/bat) |
| **tmux 세션 관리** | tmuxinator로 프로젝트별 레이아웃 관리 | [tmuxinator](https://github.com/tmuxinator/tmuxinator) |
| **tmux-fingers** | 화면의 URL/경로를 Vimium 스타일로 복사 | [tmux-fingers](https://github.com/Morantron/tmux-fingers) |
| **tmux-fzf** | fzf 기반 세션/윈도우/패널 전환기 | [tmux-fzf](https://github.com/sainnhe/tmux-fzf) |
| **tmux 테마** | Catppuccin 공식 tmux 테마 | [catppuccin/tmux](https://github.com/catppuccin/tmux) |

---

## 참고 자료

| 도구 | 공식 문서 | GitHub |
|---|---|---|
| Ghostty | [ghostty.org](https://ghostty.org/) | [ghostty-org/ghostty](https://github.com/ghostty-org/ghostty) |
| tmux | [tmux wiki](https://github.com/tmux/tmux/wiki) | [tmux/tmux](https://github.com/tmux/tmux) |
| Neovim | [neovim.io](https://neovim.io/doc/) | [neovim/neovim](https://github.com/neovim/neovim) |
| LazyVim | [lazyvim.org](https://www.lazyvim.org/) | [LazyVim/LazyVim](https://github.com/LazyVim/LazyVim) |
| yazi | [yazi-rs.github.io](https://yazi-rs.github.io/) | [sxyazi/yazi](https://github.com/sxyazi/yazi) |
| TPM | — | [tmux-plugins/tpm](https://github.com/tmux-plugins/tpm) |
| lazygit | — | [jesseduffield/lazygit](https://github.com/jesseduffield/lazygit) |
| fzf | — | [junegunn/fzf](https://github.com/junegunn/fzf) |
| starship | [starship.rs](https://starship.rs/) | [starship/starship](https://github.com/starship/starship) |

---

## Appendix: AI 자동 셋업

> **이 섹션은 Claude Code 같은 AI 에이전트가 읽고 자동으로 실행하기 위한 것이다.**
> 사용자가 Claude Code에게 "이 가이드를 읽고 TUI 환경을 셋업해줘"라고 요청하면, AI는 아래 단계를 순서대로 실행한다.

### AI 에이전트를 위한 지침

- 모든 명령은 **멱등성(idempotent)** 을 갖는다. 이미 설치/설정된 항목은 건너뛴다.
- 각 단계 실행 후 **검증 명령**으로 성공 여부를 확인한다.
- 기존 설정 파일이 있으면 **반드시 백업**한 후 덮어쓴다.
- `~/.zshrc`는 덮어쓰지 않고 **블록 단위로 추가**한다. 이미 해당 블록이 있으면 건너뛴다.
- 사용자에게 각 단계의 진행 상황을 보고한다.
- tmux 플러그인 설치(`prefix+I`)와 Neovim 첫 실행(플러그인 자동 설치)은 **사용자의 수동 실행이 필요**하므로, 마지막에 안내한다.

---

### Step 1: Homebrew 확인

```bash
# 확인
command -v brew
```

Homebrew가 없으면 설치를 안내하고 중단한다. AI가 Homebrew를 직접 설치하지 않는다.

---

### Step 2: 핵심 도구 설치

```bash
brew install ghostty tmux neovim
```

**검증:**
```bash
ghostty --version && tmux -V && nvim --version | head -1
```

사용자에게 yazi(TUI 파일 매니저) 설치 여부를 확인한다. 코딩 중 파일 탐색만 필요하면 nvim 파일 트리로 충분하므로 선택 사항이다.

```bash
# 선택: yazi 설치
brew install yazi
```

---

### Step 3: 보조 CLI 도구 설치

```bash
brew install fzf fd ripgrep bat eza zoxide lazygit starship
```

**검증:**
```bash
fzf --version && fd --version && rg --version && bat --version && eza --version && zoxide --version && lazygit --version && starship --version
```

---

### Step 4: yazi 미리보기 의존성 설치 (yazi를 설치한 경우)

```bash
brew install ffmpegthumbnailer poppler imagemagick
```

---

### Step 5: Nerd Font 설치

```bash
brew install --cask font-jetbrains-mono-nerd-font
```

**검증:**
```bash
brew list --cask | grep font-jetbrains-mono-nerd-font
```

---

### Step 6: Ghostty 설정

```bash
mkdir -p ~/.config/ghostty
```

**파일: `~/.config/ghostty/config`** — 기존 파일이 있으면 `~/.config/ghostty/config.backup.$(date +%s)`로 백업 후 작성:

```
font-family = "JetBrains Mono"
font-size = 14
theme = Catppuccin Mocha
image-storage-limit = 320000000
window-padding-x = 8
window-padding-y = 8
clipboard-read = allow
clipboard-write = allow
cursor-style = bar
cursor-style-blink = false
mouse-scroll-multiplier = 3
```

---

### Step 7: TPM (tmux Plugin Manager) 설치

```bash
# 이미 있으면 건너뜀
[ -d ~/.tmux/plugins/tpm ] || git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

**검증:**
```bash
ls ~/.tmux/plugins/tpm/tpm
```

---

### Step 8: tmux 설정

**파일: `~/.tmux.conf`** — 기존 파일이 있으면 `~/.tmux.conf.backup.$(date +%s)`로 백업 후 작성:

```bash
# ══════════════════════════════════════════════
# 블록 1: 기본 옵션
# ══════════════════════════════════════════════
unbind C-b
set -g prefix C-a
bind C-a send-prefix
bind r source-file ~/.tmux.conf \; display "Reloaded!"
set -g default-terminal "tmux-256color"
set -ag terminal-overrides ",xterm-256color:RGB"
set -g mouse on
set -sg escape-time 0
set -g focus-events on
set -g history-limit 50000
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on
set -g set-clipboard on

# ══════════════════════════════════════════════
# 블록 2: 패널(Pane) 관리
# ══════════════════════════════════════════════
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
unbind '"'
unbind %
bind c new-window -c "#{pane_current_path}"
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5
bind m resize-pane -Z
bind -n S-Left previous-window
bind -n S-Right next-window
bind s choose-tree -sZ

# ══════════════════════════════════════════════
# 블록 3: 복사 모드
# ══════════════════════════════════════════════
setw -g mode-keys vi
bind -T copy-mode-vi v send -X begin-selection
bind -T copy-mode-vi y send -X copy-pipe-and-cancel "pbcopy"

# ══════════════════════════════════════════════
# 블록 4: 팝업 바인딩 (tmux 3.2+)
# ══════════════════════════════════════════════
bind e display-popup -d "#{pane_current_path}" -w 90% -h 85% -E 'tmux new-session yazi \; set status off'
bind g display-popup -d "#{pane_current_path}" -w 90% -h 90% -E "lazygit"
bind f display-popup -d "#{pane_current_path}" -w 80% -h 80% -E \
    'file=$(fzf --preview "bat --color=always --style=numbers {}") && nvim "$file"'
bind t display-popup -d "#{pane_current_path}" -w 80% -h 80% -E "zsh"

# ══════════════════════════════════════════════
# 블록 5: 상태바
# ══════════════════════════════════════════════
set -g status-position top
set -g status-interval 5
set -g status-style "bg=#1e1e2e,fg=#cdd6f4"
set -g status-left "#[fg=#1e1e2e,bg=#89b4fa,bold] #S "
set -g status-left-length 30
setw -g window-status-format " #[fg=#6c7086]#I:#W "
setw -g window-status-current-format "#[fg=#1e1e2e,bg=#a6e3a1,bold] #I:#W "
set -g status-right "#{?client_prefix,#[bg=#f38ba8,fg=#1e1e2e,bold] PREFIX ,}#[fg=#a6e3a1] #(cd #{pane_current_path}; git branch --show-current 2>/dev/null) #[fg=#f9e2af]%H:%M "
set -g status-right-length 60
set -g pane-border-style "fg=#313244"
set -g pane-active-border-style "fg=#89b4fa"

# ══════════════════════════════════════════════
# 블록 6: 플러그인
# ══════════════════════════════════════════════
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'
set -g @plugin 'tmux-plugins/tmux-yank'
set -g @plugin 'christoomey/vim-tmux-navigator'
set -g @resurrect-strategy-nvim 'session'
set -g @resurrect-capture-pane-contents 'on'
set -g @continuum-restore 'on'
set -g @continuum-save-interval '15'

# TPM 초기화 (반드시 마지막 줄)
run '~/.tmux/plugins/tpm/tpm'
```

---

### Step 9: LazyVim 설치

```bash
# 기존 nvim 설정 백업 (있는 경우)
[ -d ~/.config/nvim ] && mv ~/.config/nvim ~/.config/nvim.backup.$(date +%s)
[ -d ~/.local/share/nvim ] && mv ~/.local/share/nvim ~/.local/share/nvim.backup.$(date +%s)
[ -d ~/.local/state/nvim ] && mv ~/.local/state/nvim ~/.local/state/nvim.backup.$(date +%s)
[ -d ~/.cache/nvim ] && mv ~/.cache/nvim ~/.cache/nvim.backup.$(date +%s)

# LazyVim starter 클론
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
```

**검증:**
```bash
ls ~/.config/nvim/init.lua
```

---

### Step 10: Neovim 커스텀 플러그인

**파일: `~/.config/nvim/lua/plugins/custom.lua`** — 새로 작성:

```lua
return {
  {
    "catppuccin/nvim",
    name = "catppuccin",
    lazy = false,
    priority = 1000,
    opts = {
      transparent_background = true,
    },
  },
  {
    "christoomey/vim-tmux-navigator",
    keys = {
      { "<C-h>", "<cmd>TmuxNavigateLeft<cr>",  desc = "Go to left pane" },
      { "<C-j>", "<cmd>TmuxNavigateDown<cr>",  desc = "Go to lower pane" },
      { "<C-k>", "<cmd>TmuxNavigateUp<cr>",    desc = "Go to upper pane" },
      { "<C-l>", "<cmd>TmuxNavigateRight<cr>", desc = "Go to right pane" },
    },
  },
  {
    "MeanderingProgrammer/render-markdown.nvim",
    ft = { "markdown", "mdx" },
  },
}
```

---

### Step 11: yazi 설정 (yazi를 설치한 경우)

```bash
mkdir -p ~/.config/yazi
```

**파일: `~/.config/yazi/yazi.toml`** — 기존 파일이 있으면 백업 후 작성:

```toml
[manager]
show_hidden = true
sort_by = "natural"
sort_dir_first = true
linemode = "size"

[preview]
max_width = 1000
max_height = 1000

[opener]
edit = [
    { run = 'nvim "$@"', block = true, desc = "Neovim" },
]

[open]
rules = [
    { mime = "text/*", use = "edit" },
    { name = "*.py",   use = "edit" },
    { name = "*.js",   use = "edit" },
    { name = "*.ts",   use = "edit" },
    { name = "*.tsx",  use = "edit" },
    { name = "*.jsx",  use = "edit" },
    { name = "*.json", use = "edit" },
    { name = "*.yaml", use = "edit" },
    { name = "*.yml",  use = "edit" },
    { name = "*.toml", use = "edit" },
    { name = "*.md",   use = "edit" },
    { name = "*.sh",   use = "edit" },
    { name = "*.lua",  use = "edit" },
    { name = "*.html", use = "edit" },
    { name = "*.css",  use = "edit" },
    { name = "*.env*", use = "edit" },
    { name = "Dockerfile*", use = "edit" },
    { name = "Makefile", use = "edit" },
]
```

**파일: `~/.config/yazi/keymap.toml`** — 기존 파일이 있으면 백업 후 작성:

```toml
[[manager.prepend_keymap]]
on   = ["<Enter>"]
run  = "open"
desc = "Open file (nvim)"

[[manager.prepend_keymap]]
on   = ["V"]
run  = '''
    shell 'tmux send-keys -t "{right}" "nvim \"$0\"" Enter' --confirm
'''
desc = "Open in nvim (right tmux pane)"

[[manager.prepend_keymap]]
on   = ["g", "d"]
run  = "cd ~/Dev"
desc = "Go to Dev directory"

[[manager.prepend_keymap]]
on   = ["g", "c"]
run  = "cd ~/.config"
desc = "Go to config directory"
```

---

### Step 12: 셸 환경 설정 (~/.zshrc)

#### Step 12-1: 프롬프트 충돌 감지

starship 프롬프트는 **선택 사항**이다. 기존 셸에 다른 프롬프트 테마(oh-my-zsh 테마, powerlevel10k 등)가 설정되어 있으면 **프롬프트가 충돌하여 깨진다.** 반드시 먼저 확인하고 사용자에게 선택을 요청한다.

```bash
# oh-my-zsh 테마 확인
grep '^ZSH_THEME=' ~/.zshrc 2>/dev/null
```

**ZSH_THEME이 비어있지 않은 값으로 설정되어 있으면 (예: `ZSH_THEME="agnoster"`, `ZSH_THEME="robbyrussell"`)**, 사용자에게 다음을 안내하고 선택을 요청한다:

> **프롬프트 충돌이 감지되었습니다.**
> 현재 oh-my-zsh 테마(`ZSH_THEME="..."`)가 설정되어 있습니다.
> starship 프롬프트와 동시에 사용하면 프롬프트가 깨집니다.
>
> 1. **기존 테마 유지 (권장)** — 기존 프롬프트를 그대로 사용합니다. starship 설정을 건너뜁니다.
> 2. **starship 사용** — `ZSH_THEME=""`로 변경합니다. starship이 git 브랜치, 언어 버전, 실행시간 등을 자동 표시합니다.

- 사용자가 **기존 테마 유지**를 선택하면: 아래 블록에서 `eval "$(starship init zsh)"` 줄을 제거하고 추가한다.
- 사용자가 **starship**을 선택하면: `ZSH_THEME=""`로 변경하고, 아래 블록을 그대로 추가한다.

#### Step 12-2: zshrc 블록 추가

아래 블록을 `~/.zshrc`에 추가한다. **기존 내용은 건드리지 않는다.** `# === TUI-IDE-SETUP ===` 마커가 이미 있으면 해당 블록을 교체한다. 없으면 파일 끝에 추가한다.

```bash
# === TUI-IDE-SETUP START ===
# 이 블록은 TUI IDE 셋업 가이드에 의해 자동 생성되었습니다.
# 수정 시 START/END 마커 사이만 편집하세요.

# 기본 에디터
export EDITOR="nvim"
export VISUAL="nvim"

# dev 스크립트 경로
export PATH="$HOME/.local/bin:$PATH"

# zoxide — cd 대체
eval "$(zoxide init zsh)"

# fzf — fuzzy finder 셸 통합
source <(fzf --zsh)

# starship — 프롬프트 (선택 사항, 기존 oh-my-zsh 테마 사용 시 제거)
# eval "$(starship init zsh)"

# bat — cat 대체
alias cat="bat"

# eza — ls 대체
alias ls="eza --icons --group-directories-first"
alias ll="eza --icons --group-directories-first -la"
alias tree="eza --icons --tree"

# lazygit
alias lg="lazygit"

# yazi wrapper: 종료 시 마지막 디렉토리로 cd
function y() {
  local tmp="$(mktemp -t "yazi-cwd.XXXXXX")" cwd
  yazi "$@" --cwd-file="$tmp"
  if cwd="$(command cat -- "$tmp")" && [ -n "$cwd" ] && [ "$cwd" != "$PWD" ]; then
    builtin cd -- "$cwd"
  fi
  rm -f -- "$tmp"
}
# === TUI-IDE-SETUP END ===
```

**마커 기반 교체 로직:**
1. `~/.zshrc`에서 `# === TUI-IDE-SETUP START ===`와 `# === TUI-IDE-SETUP END ===` 사이를 찾는다.
2. 있으면 해당 블록을 위 내용으로 교체한다.
3. 없으면 파일 끝에 위 블록을 추가한다.

---

### Step 13: dev 스크립트 설치

```bash
mkdir -p ~/.local/bin
```

**파일: `~/.local/bin/dev`** — 기존 파일이 있으면 백업 후 작성:

```bash
#!/bin/bash
# dev — 프로젝트별 tmux 개발 세션 자동 생성
# Usage: dev [project-path] [session-name]
# Example: dev ~/work/my-api api

PROJECT_PATH="${1:-.}"
PROJECT_PATH="$(cd "$PROJECT_PATH" && pwd)"
SESSION_NAME="${2:-$(basename "$PROJECT_PATH")}"

if tmux has-session -t "$SESSION_NAME" 2>/dev/null; then
    tmux attach-session -t "$SESSION_NAME"
    exit 0
fi

cd "$PROJECT_PATH" || exit 1

tmux new-session -d -s "$SESSION_NAME" -n "code" -c "$PROJECT_PATH"
tmux send-keys -t "$SESSION_NAME:code" "nvim ." C-m

tmux new-window -t "$SESSION_NAME" -n "server" -c "$PROJECT_PATH"
tmux split-window -h -t "$SESSION_NAME:server" -c "$PROJECT_PATH"

tmux new-window -t "$SESSION_NAME" -n "ai" -c "$PROJECT_PATH"
tmux split-window -v -t "$SESSION_NAME:ai" -c "$PROJECT_PATH"

tmux select-window -t "$SESSION_NAME:code"
tmux attach-session -t "$SESSION_NAME"
```

```bash
chmod +x ~/.local/bin/dev
```

---

### Step 14: 검증

모든 단계가 끝난 후 아래 명령으로 전체 설치 상태를 확인한다:

```bash
echo "=== 핵심 도구 ==="
command -v ghostty && echo "ghostty: OK" || echo "ghostty: MISSING"
command -v tmux && echo "tmux: OK" || echo "tmux: MISSING"
command -v nvim && echo "nvim: OK" || echo "nvim: MISSING"
command -v yazi && echo "yazi: OK" || echo "yazi: MISSING"

echo ""
echo "=== 보조 도구 ==="
for cmd in fzf fd rg bat eza zoxide lazygit starship; do
  command -v $cmd && echo "$cmd: OK" || echo "$cmd: MISSING"
done

echo ""
echo "=== 설정 파일 ==="
[ -f ~/.config/ghostty/config ] && echo "ghostty config: OK" || echo "ghostty config: MISSING"
[ -f ~/.tmux.conf ] && echo "tmux.conf: OK" || echo "tmux.conf: MISSING"
[ -f ~/.config/nvim/init.lua ] && echo "nvim config: OK" || echo "nvim config: MISSING"
[ -f ~/.config/nvim/lua/plugins/custom.lua ] && echo "nvim custom plugins: OK" || echo "nvim custom plugins: MISSING"
[ -f ~/.config/yazi/yazi.toml ] && echo "yazi config: OK" || echo "yazi config: MISSING"
[ -f ~/.config/yazi/keymap.toml ] && echo "yazi keymap: OK" || echo "yazi keymap: MISSING"
[ -d ~/.tmux/plugins/tpm ] && echo "TPM: OK" || echo "TPM: MISSING"
[ -x ~/.local/bin/dev ] && echo "dev script: OK" || echo "dev script: MISSING"

echo ""
echo "=== zshrc 블록 ==="
grep -q "TUI-IDE-SETUP START" ~/.zshrc && echo "zshrc block: OK" || echo "zshrc block: MISSING"
```

---

### Step 15: 사용자에게 안내할 수동 작업

AI가 실행할 수 없는 작업을 사용자에게 안내한다:

```
셋업이 완료되었습니다. 아래 작업을 수동으로 수행해주세요:

1. 터미널을 재시작하거나 `source ~/.zshrc`를 실행하세요.

2. tmux를 실행한 후 플러그인을 설치하세요:
   $ tmux
   Ctrl+a 를 누른 후 Shift+I (대문자 I)
   하단에 설치 진행 메시지가 나옵니다. 완료되면 Enter.

3. Neovim을 처음 실행하면 플러그인이 자동 설치됩니다 (1~2분 소요):
   $ nvim

4. Neovim에서 Language Server를 설치하세요:
   :LazyExtras 에서 사용하는 언어를 x로 활성화
   또는 :Mason 에서 개별 설치

5. Ghostty를 재시작하면 새 설정이 적용됩니다.
```

---

## Appendix: AI 롤백 가이드

> **이 섹션은 Claude Code 같은 AI 에이전트가 읽고 TUI 환경을 자동으로 롤백(제거)하기 위한 것이다.**
> 사용자가 "TUI 환경을 롤백해줘"라고 요청하면, AI는 아래 단계를 순서대로 실행한다.

### AI 에이전트를 위한 롤백 지침

- **롤백은 3가지 수준**으로 나뉜다. 사용자에게 어떤 수준을 원하는지 반드시 먼저 확인한다.
- 각 단계 실행 전 **삭제 대상이 존재하는지 확인**한다. 없으면 건너뛴다.
- 백업 파일(`*.backup.*`)이 있으면 **복원 여부를 사용자에게 확인**한다.
- 모든 삭제 명령 실행 전 사용자에게 **최종 확인**을 받는다.

### 롤백 수준 선택

사용자에게 아래 3가지 중 선택하게 한다:

| 수준 | 설명 | 제거 범위 |
|---|---|---|
| **Level 1: 설정만 제거** | 도구는 유지, 커스텀 설정만 초기화 | 설정 파일, zshrc 블록, dev 스크립트 |
| **Level 2: 설정 + 도구 제거** | 모든 TUI 도구와 설정을 제거 | Level 1 + brew 패키지 언인스톨 |
| **Level 3: 완전 제거** | 모든 흔적을 제거하고 백업 파일도 정리 | Level 2 + 캐시, 데이터, 백업 파일 |

---

### Level 1: 설정만 제거

도구(brew 패키지)는 그대로 두고, 이 가이드가 생성한 설정 파일만 제거한다.
도구를 유지하면서 설정을 처음부터 다시 하고 싶을 때 사용한다.

#### Rollback 1-1: zshrc 블록 제거

`~/.zshrc`에서 `# === TUI-IDE-SETUP START ===`부터 `# === TUI-IDE-SETUP END ===`까지의 블록을 제거한다. **블록 외부의 내용은 절대 건드리지 않는다.**

```bash
# 마커 블록이 있는지 확인
grep -n "TUI-IDE-SETUP" ~/.zshrc
```

마커가 확인되면 해당 블록(START 줄 ~ END 줄)을 삭제한다.

**검증:**
```bash
grep -c "TUI-IDE-SETUP" ~/.zshrc
# 0 이어야 함
```

#### Rollback 1-2: tmux 설정 제거

```bash
# 현재 실행 중인 tmux 세션 확인 — 있으면 사용자에게 알린다
tmux list-sessions 2>/dev/null

# tmux.conf 제거
rm -f ~/.tmux.conf
```

#### Rollback 1-3: Ghostty 설정 제거

```bash
rm -f ~/.config/ghostty/config
```

#### Rollback 1-4: Neovim 설정 제거 (LazyVim)

```bash
rm -rf ~/.config/nvim
```

#### Rollback 1-5: yazi 설정 제거

```bash
rm -rf ~/.config/yazi
```

#### Rollback 1-6: dev 스크립트 제거

```bash
rm -f ~/.local/bin/dev
```

#### Rollback 1-7: TPM 제거

```bash
rm -rf ~/.tmux/plugins
```

#### Rollback 1-8: 백업 파일 복원

아래 위치에 백업 파일이 있는지 확인한다. 있으면 사용자에게 복원 여부를 묻는다.

```bash
echo "=== 백업 파일 검색 ==="
ls -d ~/.tmux.conf.backup.* 2>/dev/null
ls -d ~/.config/ghostty/config.backup.* 2>/dev/null
ls -d ~/.config/nvim.backup.* 2>/dev/null
ls -d ~/.local/share/nvim.backup.* 2>/dev/null
ls -d ~/.local/state/nvim.backup.* 2>/dev/null
ls -d ~/.cache/nvim.backup.* 2>/dev/null
ls -d ~/.config/yazi.backup.* 2>/dev/null
ls -d ~/.local/bin/dev.backup.* 2>/dev/null
```

사용자가 복원을 원하면 가장 최신 백업(타임스탬프가 가장 큰 것)을 원래 경로로 이동한다:

```bash
# 예시: nvim 설정 복원 (가장 최신 백업)
LATEST=$(ls -d ~/.config/nvim.backup.* 2>/dev/null | sort -t. -k3 -n | tail -1)
[ -n "$LATEST" ] && mv "$LATEST" ~/.config/nvim
```

---

### Level 2: 설정 + 도구 제거

Level 1의 모든 단계를 먼저 실행한 후, 아래를 추가로 실행한다.

#### Rollback 2-1: 보조 CLI 도구 제거

```bash
brew uninstall starship lazygit eza bat zoxide fzf ripgrep fd
```

#### Rollback 2-2: yazi 미리보기 의존성 제거

```bash
brew uninstall ffmpegthumbnailer poppler imagemagick
```

#### Rollback 2-3: 핵심 도구 제거

```bash
brew uninstall yazi neovim tmux ghostty
```

#### Rollback 2-4: Nerd Font 제거

```bash
brew uninstall --cask font-jetbrains-mono-nerd-font
```

**검증:**
```bash
echo "=== 제거 확인 ==="
for cmd in ghostty tmux nvim yazi fzf fd rg bat eza zoxide lazygit starship; do
  command -v $cmd 2>/dev/null && echo "$cmd: 아직 남아있음" || echo "$cmd: 제거됨"
done
```

---

### Level 3: 완전 제거

Level 1 + Level 2의 모든 단계를 먼저 실행한 후, 아래를 추가로 실행한다.

#### Rollback 3-1: 캐시 및 데이터 파일 제거

```bash
# Neovim 데이터/캐시
rm -rf ~/.local/share/nvim
rm -rf ~/.local/state/nvim
rm -rf ~/.cache/nvim

# tmux 플러그인 디렉토리 (빈 디렉토리 포함)
rm -rf ~/.tmux

# tmux resurrect 저장 데이터
rm -rf ~/.local/share/tmux

# lazygit 설정/상태
rm -rf ~/.config/lazygit
rm -rf ~/Library/Application\ Support/lazygit

# bat 캐시
rm -rf ~/.cache/bat
rm -rf ~/.config/bat

# starship 설정
rm -f ~/.config/starship.toml

# zoxide 데이터베이스
rm -f ~/.local/share/zoxide/db.zo

# yazi 캐시
rm -rf ~/.cache/yazi

# Ghostty 설정 디렉토리 (빈 디렉토리 포함)
rmdir ~/.config/ghostty 2>/dev/null

# dev 스크립트 디렉토리 (비어있으면 제거)
rmdir ~/.local/bin 2>/dev/null
```

#### Rollback 3-2: 백업 파일 제거

```bash
rm -f ~/.tmux.conf.backup.*
rm -rf ~/.config/ghostty/config.backup.*
rm -rf ~/.config/nvim.backup.*
rm -rf ~/.local/share/nvim.backup.*
rm -rf ~/.local/state/nvim.backup.*
rm -rf ~/.cache/nvim.backup.*
rm -rf ~/.config/yazi.backup.*
rm -f ~/.local/bin/dev.backup.*
```

---

### 롤백 후 검증

모든 롤백이 끝난 후 아래 명령으로 잔여 항목을 확인한다:

```bash
echo "=== 잔여 설정 파일 ==="
[ -f ~/.config/ghostty/config ] && echo "ghostty config: 남아있음" || echo "ghostty config: 없음"
[ -f ~/.tmux.conf ] && echo "tmux.conf: 남아있음" || echo "tmux.conf: 없음"
[ -d ~/.config/nvim ] && echo "nvim config: 남아있음" || echo "nvim config: 없음"
[ -d ~/.config/yazi ] && echo "yazi config: 남아있음" || echo "yazi config: 없음"
[ -d ~/.tmux/plugins ] && echo "tmux plugins: 남아있음" || echo "tmux plugins: 없음"
[ -x ~/.local/bin/dev ] && echo "dev script: 남아있음" || echo "dev script: 없음"
grep -q "TUI-IDE-SETUP" ~/.zshrc 2>/dev/null && echo "zshrc block: 남아있음" || echo "zshrc block: 없음"

echo ""
echo "=== 잔여 백업 파일 ==="
ls -d ~/.tmux.conf.backup.* ~/.config/ghostty/config.backup.* ~/.config/nvim.backup.* ~/.local/share/nvim.backup.* ~/.local/state/nvim.backup.* ~/.cache/nvim.backup.* ~/.config/yazi.backup.* ~/.local/bin/dev.backup.* 2>/dev/null || echo "백업 파일 없음"

echo ""
echo "=== 도구 설치 상태 ==="
for cmd in ghostty tmux nvim yazi fzf fd rg bat eza zoxide lazygit starship; do
  command -v $cmd 2>/dev/null && echo "$cmd: 설치됨" || echo "$cmd: 없음"
done
```

### 사용자에게 안내할 수동 작업

```
롤백이 완료되었습니다. 아래 작업을 수동으로 수행해주세요:

1. 터미널을 재시작하거나 `source ~/.zshrc`를 실행하세요.
2. Ghostty를 재시작하면 설정이 초기화됩니다.
3. Ghostty를 제거한 경우, 기존에 사용하던 터미널 앱(iTerm2, Terminal.app 등)으로 전환하세요.
4. Nerd Font를 제거한 경우, 터미널 폰트를 시스템 기본 폰트로 변경하세요.
```
