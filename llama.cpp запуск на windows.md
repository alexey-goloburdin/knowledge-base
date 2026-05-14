# Запуск llama.cpp на винде (без MTP)
  
Для qwen3.6-27b:  

```powershell
C:\soft_portable\llama\llama-server.exe `
  -m C:\Users\sterx\.lmstudio\models\lmstudio-community\Qwen3.6-27B-GGUF\Qwen3.6-27B-Q4_K_M.gguf `
  --host 0.0.0.0 --port 8080 `
  --jinja `
  -ngl 99 `
  -ngld 99 `
  -c 16384 `
  --temp 0.6 `
  --top-p 0.95 `
  --top-k 20 `
  --min-p 0.0 `
  --presence-penalty 0.0 `
  --repeat-penalty 1.0
```
  
Для qwen3.6-35b-a3b:  

```powershell
C:\soft_portable\llama\llama-server.exe `
   -m C:\Users\sterx\.lmstudio\models\lmstudio-community\Qwen3.6-35B-A3B-GGUF\Qwen3.6-35B-A3B-Q4_K_M.gguf `
   --host 0.0.0.0 --port 8080 `
   --jinja `
   -ngl 99 `
   -ngld 99 `
   -c 16384 `
   --temp 0.6 `
   --top-p 0.95 `
   --top-k 20 `
   --min-p 0.0 `
   --presence-penalty 0.0 `
   --repeat-penalty 1.0
```