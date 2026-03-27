## 1. Overview

I worked on setting up RamaLama and understanding how it simplifies running AI models locally.

The goal of this task was not just to run a model, but to:

understand how RamaLama works
try different model sources (Ollama, Hugging Face)
observe failures and debug them
compare different approaches

During this process, I ended up exploring:

container-based vs non-container execution
model size vs system limitations
storage constraints and cleanup

## 2. My Setup

I ran everything on:

- MacBook Air (Apple Silicon)
- CPU only (no GPU support)
- Podman as the container engine (default in RamaLama)

While running commands, I consistently got this warning:

"applehv does not support GPU, only libkrun supports GPU"

So all experiments were done in a CPU-only environment, which influenced performance and failures later.

## 3. Installing RamaLama

I installed RamaLama using the official install script.

The installation handled:

- dependencies (llama.cpp, ggml, etc.)
- environment setup
- binary installation

Everything completed successfully.

<img width="808" height="879" alt="Installation_SS" src="https://github.com/user-attachments/assets/8309b1b6-f881-43b7-81d5-b5e5fda30d39" />

## 4. Verifying Installation

To confirm everything was set up correctly, I checked:

version
available commands

This also helped me understand what RamaLama can do (run, pull, inspect, etc.)

<img width="1455" height="60" alt="Ramalama_version" src="https://github.com/user-attachments/assets/f2f19c86-e809-425f-8e20-3a09f981eefb" />

<img width="1454" height="110" alt="Ramalama_help" src="https://github.com/user-attachments/assets/19ee30a1-54dd-4a56-9cde-4ece4108682a" />


## 5. First Model (Ollama – Successful Run)

I started with a smaller model using the Ollama transport:

ramalama run ollama://tinyllama "What are the Four Foundations of the Fedora project?"

This worked successfully.

Observations:

Model downloaded quickly (~600MB)
Response was generated without issues
Output was readable but not fully accurate (expected for a small model)

<img width="1459" height="367" alt="ollama_tinyllama" src="https://github.com/user-attachments/assets/f177f814-1ff2-4b5a-88e9-8eeb503f8723" />

## 6. Trying Larger Models (Hugging Face – Failure Case)

Next, I tried a larger model:

huggingface://ibm-granite/granite-3.3-2b-instruct

This failed during execution.

Error:

container health check timed out
model did not start successfully

<img width="1456" height="141" alt="Hugging face" src="https://github.com/user-attachments/assets/52f33d46-6b4a-4a3b-af41-29384ec06fc7" />

What I understood

This failure was likely due to:

- large model size (~4–5GB)
- CPU-only execution
- container overhead

This showed that: Not all models are practical for all systems

## 7. Storage Issues While Downloading Models

While experimenting with another model (Qwen / Phi), I encountered:

OSError: No space left on device

<img width="1465" height="117" alt="No Space" src="https://github.com/user-attachments/assets/4078deb0-55bd-4081-9c4e-7deb8556aae1" />


What I did

I checked disk usage:

<img width="1066" height="44" alt="Disk Space" src="https://github.com/user-attachments/assets/22abc573-b85f-4e33-8d30-30fdd0333fa0" />


It showed limited available storage.

## 8. Fixing the Storage Problem

To fix this, I cleaned unused containers and images:

podman system prune -a

<img width="1449" height="311" alt="Cleanup" src="https://github.com/user-attachments/assets/950ce229-02fc-41da-a32d-46dad5de06d7" />

This freed space and allowed me to continue testing.

Insight

This step made me realize:

-> Running local AI models is resource heavy, especially storage.

## 9. Second Attempt (HuggingFace – Still Failing)

Even after fixing storage, some models still failed due to:

- container startup timeout
- CPU limitations

So I tried a different approach.

## 10. Switching Strategy (Key Turning Point)

Instead of running models inside containers, I used:

--nocontainer --runtime mlx

This runs the model directly using Apple Silicon optimized runtime.

## 11. HuggingFace Model (Successful with MLX)

Command:

ramalama --nocontainer --runtime mlx run huggingface://TinyLlama/TinyLlama-1.1B-Chat-v1.0 ...

This worked successfully.

<img width="1470" height="244" alt="Succes_model_las" src="https://github.com/user-attachments/assets/63d94a69-0f0a-40fa-bb8c-882975e0f33c" />

What changed?

- No container overhead
- Better compatibility with Mac (M-series chips)
- Faster and more stable execution

## 12. Additional Exploration (Model Inspection)

I also explored:

ramalama inspect ollama://tinyllama

This showed:

model format
metadata
storage path

<img width="1470" height="165" alt="Optional" src="https://github.com/user-attachments/assets/7d384e0b-9716-431f-b667-e7ed04666023" />

## 13. Comparison of Approaches

From all experiments:

Ollama (Container-based)
✅ Easy to use
✅ Works reliably for small models
❌ Limited flexibility

Hugging Face (Container-based)
❌ Failed for larger models
❌ Heavy resource usage
❌ Container startup issues

Hugging Face (MLX – No container)
✅ Worked on Mac
✅ More stable
✅ Better suited for CPU/M-series chips

## 14. Key Learnings

Model size matters a lot
Containers add overhead
Storage is a real constraint
Different runtimes behave very differently
Debugging is part of the process

## 15. Does RamaLama make AI “boring”?

In a good way —> yes.

RamaLama simplifies:

pulling models
running them
managing them

All with a single command.

But:

-> When things fail, we still need to understand:

system limits
runtimes
storage
model sizes

So it makes the easy parts boring, but keeps the interesting parts real.

Also this drive has the whole code and outputs -> https://docs.google.com/document/d/12CrfUTpH9R2tFeSk6AE6oMJDFlPz2sVacF9_EKtTJp0/edit?usp=sharing
