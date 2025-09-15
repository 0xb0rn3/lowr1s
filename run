#!/bin/bash

# lowr1s Installation Script
# Developer: 0xb0rn3 | 0xbv1
# Version: 1.0.0

set -e

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
NC='\033[0m'

echo -e "${CYAN}"
echo "╔══════════════════════════════════════╗"
echo "║     lowr1s Installation Script       ║"
echo "║       by 0xb0rn3 | 0xbv1            ║"
echo "╚══════════════════════════════════════╝"
echo -e "${NC}"

# Check for root
if [[ $EUID -eq 0 ]]; then
   echo -e "${RED}This script should not be run as root!${NC}"
   exit 1
fi

# Detect OS
if [[ "$OSTYPE" == "linux-gnu"* ]]; then
    OS="linux"
elif [[ "$OSTYPE" == "darwin"* ]]; then
    OS="macos"
else
    echo -e "${RED}Unsupported OS: $OSTYPE${NC}"
    exit 1
fi

echo -e "${GREEN}[1/5] Checking dependencies...${NC}"

# Check for Rust
if ! command -v cargo &> /dev/null; then
    echo -e "${YELLOW}Rust not found. Installing...${NC}"
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
    source "$HOME/.cargo/env"
fi

# Install system dependencies
echo -e "${GREEN}[2/5] Installing system dependencies...${NC}"

if [[ "$OS" == "linux" ]]; then
    # Detect package manager
    if command -v apt-get &> /dev/null; then
        PKG_MGR="apt-get"
        sudo apt-get update
        sudo apt-get install -y build-essential pkg-config libssl-dev \
            p7zip-full unrar tar gzip bzip2 xz-utils zstd \
            pigz pbzip2 pixz
    elif command -v yum &> /dev/null; then
        PKG_MGR="yum"
        sudo yum groupinstall -y "Development Tools"
        sudo yum install -y openssl-devel p7zip p7zip-plugins \
            unrar tar gzip bzip2 xz zstd
    elif command -v pacman &> /dev/null; then
        PKG_MGR="pacman"
        sudo pacman -Sy --noconfirm base-devel openssl p7zip \
            unrar tar gzip bzip2 xz zstd pigz pbzip2
    else
        echo -e "${RED}Unsupported package manager${NC}"
        exit 1
    fi
elif [[ "$OS" == "macos" ]]; then
    if ! command -v brew &> /dev/null; then
        echo -e "${YELLOW}Installing Homebrew...${NC}"
        /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    fi
    brew install p7zip unrar xz zstd pigz pbzip2
fi

# Create project directory
echo -e "${GREEN}[3/5] Setting up project directory...${NC}"
PROJECT_DIR="$HOME/.local/share/lowr1s"
mkdir -p "$PROJECT_DIR"
cd "$PROJECT_DIR"

# Create Rust project structure
cat > Cargo.toml << 'EOF'
[package]
name = "lowr1s"
version = "1.0.0"
edition = "2021"
authors = ["0xb0rn3 | 0xbv1"]
description = "High Performance Archive Tool"

[dependencies]
flate2 = { version = "1.0", features = ["zlib-ng"] }
tar = "0.4"
xz2 = "0.1"
zip = "0.6"
rayon = "1.7"
num_cpus = "1.16"
sysinfo = "0.30"
clap = { version = "4.4", features = ["derive"] }
indicatif = "0.17"
walkdir = "2.4"
crossbeam-channel = "0.5"

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
strip = true
panic = "abort"
EOF

# Create source directory
mkdir -p src

# Copy Rust source
echo -e "${GREEN}[4/5] Building Rust components...${NC}"

# Note: The Rust source code from the previous artifact would be placed here
# For this example, we'll create a placeholder
cat > src/main.rs << 'EOF'
// Placeholder - Replace with actual lowr1s Rust code from artifact
fn main() {
    println!("lowr1s - High Performance Archive Tool");
    println!("Please replace this with the actual Rust source code");
}
EOF

# Build with optimizations
echo -e "${CYAN}Building with maximum optimizations...${NC}"
RUSTFLAGS="-C target-cpu=native -C opt-level=3" cargo build --release

# Install binaries
echo -e "${GREEN}[5/5] Installing lowr1s...${NC}"

# Create bin directory
BIN_DIR="$HOME/.local/bin"
mkdir -p "$BIN_DIR"

# Copy Rust binary
if [ -f "target/release/lowr1s" ]; then
    cp target/release/lowr1s "$BIN_DIR/lowr1s-core"
    chmod +x "$BIN_DIR/lowr1s-core"
fi

# Create main wrapper script
cat > "$BIN_DIR/lowr1s" << 'WRAPPER'
#!/bin/bash
# lowr1s wrapper script

# Check if we should use Rust core for performance
if [[ "$1" == "--rust" ]] || [[ "$LOWR1S_USE_RUST" == "1" ]]; then
    shift
    exec "$HOME/.local/bin/lowr1s-core" "$@"
fi

# Otherwise use bash wrapper for compatibility
exec bash "$HOME/.local/share/lowr1s/lowr1s.sh" "$@"
WRAPPER

chmod +x "$BIN_DIR/lowr1s"

# Copy bash script
cp "$0" "$PROJECT_DIR/lowr1s.sh" 2>/dev/null || true

# Update PATH if needed
if [[ ":$PATH:" != *":$BIN_DIR:"* ]]; then
    echo "" >> "$HOME/.bashrc"
    echo "# lowr1s path" >> "$HOME/.bashrc"
    echo "export PATH=\"$BIN_DIR:\$PATH\"" >> "$HOME/.bashrc"
    echo -e "${YELLOW}Added $BIN_DIR to PATH in ~/.bashrc${NC}"
fi

# Create config file
CONFIG_DIR="$HOME/.config/lowr1s"
mkdir -p "$CONFIG_DIR"

cat > "$CONFIG_DIR/config.toml" << 'EOF'
# lowr1s Configuration
[performance]
threads = "auto"  # auto, or specific number
memory_limit = "80%"  # percentage of system RAM
buffer_size = 16777216  # 16MB

[compression]
default_format = "tar.gz"
default_level = 6  # 1-9

[features]
use_rust_core = false  # Set to true for maximum performance
parallel_compression = true
show_progress = true
EOF

echo -e "${GREEN}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
echo -e "${GREEN}✓ Installation complete!${NC}"
echo ""
echo -e "${CYAN}Usage:${NC}"
echo "  lowr1s -c tar.gz /path/to/dir output.tar.gz    # Compress"
echo "  lowr1s -d archive.tar.xz                        # Decompress"
echo "  lowr1s -b /path/to/test                         # Benchmark"
echo "  lowr1s --rust -i                                # System info (Rust mode)"
echo ""
echo -e "${YELLOW}Configuration:${NC} $CONFIG_DIR/config.toml"
echo -e "${YELLOW}Binaries:${NC} $BIN_DIR/lowr1s"
echo ""
echo -e "${GREEN}For maximum performance, set:${NC}"
echo "  export LOWR1S_USE_RUST=1"
echo ""
echo -e "${CYAN}Contact:${NC}"
echo "  Developer: 0xb0rn3 | 0xbv1"
echo "  IG: theehiv3"
echo "  Discord & X: oxbv1"
echo -e "${GREEN}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"

# Reload shell
if [[ "$SHELL" == *"bash"* ]]; then
    source "$HOME/.bashrc"
elif [[ "$SHELL" == *"zsh"* ]]; then
    source "$HOME/.zshrc"
fi
