# k8sgpt with local-ai in 5 steps
Evaluates the use of k8sgpt with a locally running llm using local-ai.
Tested on Apple M2 (16GB RAM; macos Sonoma). Deploys k8sgpt, gpt4all, and chaos-mesh.

## Prerequisites
* Install k8sgpt with homebrew https://docs.k8sgpt.ai/getting-started/installation/
* Build local-ai from source on Mac M1/M2 https://localai.io/basics/build/ (docker image is not working as it is not available for apple chips yet or I am missing something...)

## Steps
1. Download a model from gpt4all (best tested the one from mistral)
```
wget https://gpt4all.io/models/ggml-gpt4all-j.bin -O models/ggml-gpt4all-j
wget https://gpt4all.io/models/gguf/mistral-7b-openorca.gguf2.Q4_0.gguf -O models/mistral-7b-openorca.gguf2.Q4_0.gguf
```
2. Start local-ai server in a terminal
```
./local-ai --models-path=./models/ --debug=true
```
3. Open a second terminal. Add k8sgpt auth for localai model 
```
k8sgpt auth add --backend=localai --baseurl=http://localhost:8080/v1 --model mistral-7b-openorca.gguf2.Q4_0.gguf
```

4. Deploy chaos-mesh, deploy a single nginx pod and simulate pod-failure
```
curl -sSL https://mirrors.chaos-mesh.org/v2.6.3/install.sh | bash
kubectl create ns chaos-test
kubectl create deploy nginx --image=nginx -n chaos-test
kubectl apply -f chaos-mesh/pod-failure.yaml
```
5. View k8sgpt analysis
```
k8sgpt analyze --explain -b localai                      

 100% |██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| (1/1, 4 it/min)         
AI Provider: localai

0 chaos-test/nginx-7854ff8877-glgzc(Deployment/nginx)
- Error: back-off 1m20s restarting failed container=nginx pod=nginx-7854ff8877-glgzc_chaos-test(96e0a86d-8a14-4c1c-ab5b-686e24a5d0c6)
- Error: the last termination reason is Error container=nginx pod=nginx-7854ff8877-glgzc


Error: The nginx container in the nginx-7854ff8877-glgzc pod has failed and is being restarted.
Solution: 
1. Check the nginx container logs for any errors or issues.
2. Ensure that the nginx container image is up-to-date and compatible with the Kubernetes version.
3. Verify that the nginx pod has sufficient resources allocated (CPU and memory).
4. Check the Kubernetes configuration for any issues with the nginx deployment.
5. If the issue persists, consider increasing the back-off time to allow more time for the container to recover.<|im_end|>
```
## Misc
Tool for interacting with llm : https://llm.datasette.io/en/stable/

Plugin for downloading gpt4all models : https://github.com/simonw/llm-gpt4all
