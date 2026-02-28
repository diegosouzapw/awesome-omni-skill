---
name: add_hashtags
description: Appends relevant hashtags to journal entries based on content analysis. Intended for LLM execution.
---

# Add Hashtags To News Links

This skill is designed for an AI to parse news headlines/links and append relevant hashtags. It cannot be run as a simple script; it requires semantic understanding.

## Execution Instructions for AI
1.  Read the target file (usually a journal page or `TempLinks.md`).
2.  For each line containing a link, analyze the title and content.
3.  Append relevant hashtags to the end of the line, space-separated.
4.  **Do NOT remove existing tags.**
5.  **Use English for most tags unless specified.**

## Tagging Rules

### Entities & Stocks
- **US Stocks**: Use Ticker symbol. **Do NOT use company name.**
    - Tesla -> `#TSLA` (NOT `#Tesla`)
    - Google/Alphabet -> `#GOOG` (NOT `#Google`)
    - Microsoft -> `#MSFT`
    - Amazon -> `#AMZN`
    - Apple -> `#AAPL`
    - Meta -> `#META`
    - NVIDIA -> `#NVDA`
    - AMD -> `#AMD`
    - Intel -> `#INTC`
    - Oracle -> `#ORCL`
    - Softbank -> `#SFTBY` (NOT `#Softbank`)
    - Bloom Energy -> `#BE`
    - Applied Materials (應材) -> `#AMAT`
    - ASML (艾司摩爾) -> `#ASML`
    - SanDisk -> `#SNDK`
    - Cisco (思科) -> `#CSCO`
    - Coinbase -> `#COIN`
    - Cloudflare -> `#NET`
    - Spotify -> `#SPOT`
    - ARM -> `#ARM`
    - Micron (美光) -> `#MU`
    - Walmart (沃爾瑪) -> `#WMT`
    - Alibaba (阿里巴巴) -> `#BABA`
    - Western Digital (威騰電子) -> `#WDC`
    - Estee Lauder (雅詩蘭黛) -> `#EL`
    - Infosys -> `#INFY`
- **Taiwan Stocks**:
    - TSMC (台積電) -> `#TSM`
    - MediaTek (聯發科) -> `#MTK`
    - Acer (宏碁) -> `#宏碁`
    - Global Unichip / Alchip (創意) -> `#創意`
    - Others: Use Traditional Chinese Name (e.g., `#鴻海`, `#智邦`, `#中美晶`, `#創見`). **Do NOT use `#stock` tag for these.**
- **People**: `#Elon`, `#Jensen`, `#Trump`, `#Altman`.

### Topics & Industries
- **AI**: `#AI` (Always add if related to Artificial Intelligence, LLMs, Agents).
- **Semiconductor**: `#semicon`, `#Chips`, `#CoWoS`, `#ASIC`.
- **Crypto**: `#Crypto`, `#BTC`, `#ETH`, `#MSTR`, `#ETF`, `#Stablecoin`, `#CBDC`.
- **Finance**: `#stock` (US/Global only, NOT for TW stocks), `#Gold`, `#Silver`, `#Fed`. **Do NOT use `#Finance` unless no other tags apply.**
- **Automotive**: `#EV`, `#FSD`, `#Robotaxi`, `#Waymo`.
- **Robots**: `#humanoid`, `#robotics`.
- **Materials**: `#REE` (Rare Earth Elements), `#lithium`.
- **Energy**: `#Solar`, `#power`, `#battery`.
- **Science**: `#science`, `#universe`, `#biology`.

### Regions
- **Taiwan**: `#TW` (Add if related to Taiwan companies, politics, or economy).
- **China**: `#China` (Add if related to Chinese companies or economy).
- **Japan**: `#Japan`
- **India**: `#India`
- **Korea**: `#Korea`
- **Europe**: `#Europe`

### Specific Models/Products
- `#ChatGPT`, `#Claude`, `#Gemini`, `#Sora`, `#Llama`.
- `#iPhone`, `#Mac`, `#VisionPro`.
- `#Starlink`, `#SpaceX`.

## Example
**Input:**
`- [台積電高雄廠導入 2nm 製程，預計 2025 量產](...)`
`- [OpenAI 發布 GPT-5，全面超越人類智商](...)`

**Output:**
`- [台積電高雄廠導入 2nm 製程，預計 2025 量產](...) #TSM #TW #semicon`
`- [OpenAI 發布 GPT-5，全面超越人類智商](...) #OpenAI #AI #GPT5`
