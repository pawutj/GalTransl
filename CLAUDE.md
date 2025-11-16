# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GalTransl is an automated translation tool for visual novels (galgames) that leverages LLMs like GPT-4, Claude, DeepSeek, and local models (Sakura). It provides a complete pipeline: script extraction → translation → script injection, with advanced features like GPT dictionaries, context-aware translation, and automatic quality checking.

## Development Environment

### Setup
```bash
# Install dependencies using Poetry (recommended)
pip install poetry
poetry install
poetry shell

# Or use requirements.txt
pip install -r requirements.txt
```

### Running the Program
```bash
# Interactive mode (recommended for development)
python run_GalTransl.py

# Command-line mode
python -m GalTransl -p <project_path> -t <translator>

# Example
python -m GalTransl -p ./sampleProject -t ForGal-json
```

### Common Commands
- **Run translation**: `python run_GalTransl.py` (interactive) or `python -m GalTransl -p <path> -t <translator>`
- **Test configuration**: Start with `sampleProject` folder
- **View plugins**: Run with `-t show-plugs`
- **Generate name dictionary**: Run with `-t dump-name`

## Architecture

### Core Components

**Entry Points**:
- `run_GalTransl.py` - User-facing interactive CLI (uses InquirerPy for menus)
- `GalTransl/__main__.py` - Core worker function and argparse CLI

**Main Pipeline** (`Runner.py`):
1. Load configuration (`ConfigHelper.py`)
2. Initialize plugins (file + text plugins from `plugins/`)
3. Setup translation backend (proxy pool, token pool)
4. Execute translation via `Frontend/LLMTranslate.py`
5. Save results with atomic writes

**Translation Backends** (`Backend/`):
- `GPT4TranslateNew.py` - OpenAI/Claude/DeepSeek compatible APIs
- `SakuraTranslate.py` - Local Sakura models
- `ForGalTranslate.py` - Galgame-optimized translation (JSON input)
- `ForNovelTranslate.py` - Novel translation (no name field)
- `GenDic.py` - Automatic GPT dictionary generation
- `RebuildTranslate.py` - Cache/result rebuilding

**Data Structures**:
- `CSentense.py` - Core sentence representation (`CSentense`, `CTransList`)
- `Cache.py` - Translation cache management with atomic writes
- `Dictionary.py` - Multi-tier dictionary system (pre/post/GPT/conditional)

**Support Systems**:
- `COpenAI.py` - OpenAI API client with token pool & endpoint rotation
- `ConfigHelper.py` - YAML config parsing and validation
- `GTPlugin.py` - Plugin base classes (`GTextPlugin`, `GFilePlugin`)
- `CSplitter.py` - File splitting strategies for parallel processing
- `Name.py` - Character name handling and extraction

### Plugin System

**Plugin Types**:
1. **File Plugins** (`file_*`) - Handle different input/output formats:
   - `file_galtransl_json` - Standard JSON format (default)
   - `file_subtitle_srt_lrc_vtt` - Subtitle formats
   - `file_epub_epub` - EPUB novels
   - `file_plaintext_txt` - Plain text
   - `file_i18n_json` - i18n/mtool JSON

2. **Text Plugins** (`text_*`) - Pre/post-process text:
   - `text_common_normalfix` - Common text fixes (must run first)
   - `text_common_skipNoJP` - Skip non-Japanese sentences
   - `text_bgi_fixruby` - BGI engine ruby text fixes

**Plugin Structure**:
```
plugins/
  plugin_name/
    plugin_name.yaml  # Configuration & metadata
    plugin_name.py    # Implementation (inherits GTextPlugin/GFilePlugin)
```

**Loading Order**: File plugins process input/output, text plugins run in specified order during translation.

### Configuration System

**Config File** (`config.yaml`):
- `backendSpecific` - API keys, endpoints, model settings
- `plugin` - File & text plugin selection
- `common` - Core settings (batch size, workers, language, retry logic)
- `proxy` - Proxy configuration
- `dictionary` - Pre/post/GPT dictionary files
- `problemAnalyze` - Quality check rules

**Key Settings**:
- `gpt.numPerRequestTranslate` (default: 10) - Sentences per API call
- `workersPerProject` (default: 16) - Parallel file workers
- `gpt.contextNum` (default: 8) - Context sentences for coherence
- `splitFile` - Single file splitting (Num/Equal/no)
- `retranslKey` - Patterns to re-translate on restart

### Translation Workflow

1. **Input Processing**:
   - File plugin reads input (e.g., JSON with `name`/`message` fields)
   - Text plugins normalize/fix text
   - Pre-translation dictionary applies

2. **Translation**:
   - Sentences batched (size: `gpt.numPerRequestTranslate`)
   - Context from previous sentences included
   - GPT dictionary entries auto-injected when relevant terms detected
   - Results cached atomically after each batch

3. **Post-Processing**:
   - Post-translation dictionary applies
   - Problem analysis runs (残留日文, 词频过高, etc.)
   - Results saved to cache + output

4. **Cache System**:
   - Atomic writes prevent corruption on crashes
   - Cache format includes `pre_jp`, `post_jp`, `pre_zh`, `proofread_zh`, `problem`
   - Re-runs skip cached sentences unless `retranslKey` matches

### Dictionary System

**Three-Tier System**:

1. **Pre-translation Dictionary** (`Dict/01H字典_矫正_译前.txt`):
   - Normalizes text before translation
   - Format: `source_word[TAB]replacement_word`

2. **GPT Dictionary** (`Dict/GPT字典.txt`, project-specific):
   - Context fed to LLM only when term appears in sentence
   - Format: `source[TAB]translation[TAB]note`
   - Used for character names, settings, terminology
   - Example: `フラン[TAB]Flan[TAB]name, lady, teacher`

3. **Post-translation Dictionary** (`Dict/00通用字典_译后.txt`):
   - Simple replacement after translation
   - Conditional dictionary supports complex rules:
     - Format: `pre_jp/post_jp[TAB]condition[TAB]search[TAB]replace`
     - Conditions: `!word` (if not present), `word1[or]word2`, `word1[and]word2`

**Loading Priority**: Project dictionaries override general dictionaries.

### Parallel Processing

**Multi-Level Parallelism**:
- `workersPerProject` - Process N files simultaneously
- `splitFile` + `splitFileNum` - Split large files into chunks
- `splitFileCrossNum` - Overlap sentences between chunks for context

**Splitting Strategies**:
- `DictionaryCountSplitter` (Num) - Split every N sentences
- `EqualPartsSplitter` (Equal) - Split file into N equal parts
- Cache alignment: Splitting settings must match for cache hits

## Important Patterns

### Error Handling
- API errors trigger `apiErrorWait` delay (auto-adapting or fixed)
- `smartRetry` enables halving batch size and clearing context on parse failures
- Failed translations marked with `(Failed)` in cache
- Use `retranslFail: true` to retry failed sentences

### Translation Quality
- **GPT Dictionary is Critical**: Define all character names with settings (gender, role, age)
- **Context Matters**: Adjust `gpt.contextNum` for coherence vs. token usage
- **Problem Analysis**: Enable relevant checks in `problemAnalyze.problemList`
- **Batch Size**: Smaller batches (`gpt.numPerRequestTranslate: 3-5`) reduce errors but slower

### Cache Management
- Cache files in `transl_cache/` mirror structure of `gt_input/`
- To re-translate: Delete cache entry (no blank lines!)
- To rebuild output only: Use `rebuildr` translator
- To rebuild cache + output: Use `rebuilda` translator

### Adding New Translation Backends
1. Create `Backend/MyBackend.py` inheriting `BaseTranslate`
2. Implement `batch_translate()` method
3. Add entry to `TRANSLATOR_SUPPORTED` in `__init__.py`
4. Register in `Frontend/LLMTranslate.py` mapping

### Plugin Development
1. Copy `plugins/text_example_nouse/` as template
2. Inherit `GTextPlugin` or `GFilePlugin`
3. Implement `gtp_init()`, `before_process()`/`process_input()`, `after_process()`/`process_output()`
4. Create `.yaml` with metadata and settings
5. Add to `plugin.textPlugins` or `plugin.filePlugin` in config

## File Structure Notes

- **Input**: `gt_input/` (default) - Source files (JSON/text)
- **Output**: `gt_output/` (default) - Translated results
- **Cache**: `transl_cache/` - Translation cache (JSON)
- **Config**: `config.yaml` in project directory
- **Dictionaries**: `Dict/` (global) + project-local dictionaries
- **Plugins**: `plugins/` (global) + `<project>/plugins/` (local)

## Testing & Debugging

- Set `loggingLevel: debug` in config for verbose output
- Enable `saveLog: true` to write logs to `GalTransl.log`
- Test with small sample first (translate 1 file, check in-game display)
- Use `dump-name` translator to extract character names for dictionary building

## Version & Updates

- Current version tracked in `GalTransl/__init__.py` (`GALTRANSL_VERSION`)
- Auto-update check on startup (uses GitHub releases API)
- Windows: Shortcuts auto-generated in project folder for quick re-runs
