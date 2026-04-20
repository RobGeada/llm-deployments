# Prereqs
* OpenShift AI
* At least one `g6.12xlarge` node
* An existing Claude installation

## Deploy the Qwen3.6-35B model
```shell
PROJECT=model-namespace
oc apply -f Qwen3_6-35B-A3B-FP8.yaml -n $PROJECT
oc apply -f ../model-token-service-account -n $PROJECT
```
(This might take a while)


## Add the Claude Code certs to your local environment

```shell
# clear existing vertex settings, if you're using enterprise claude
export CLAUDE_CODE_USE_VERTEX=0 

# set up routing to your hosted model
export ANTHROPIC_AUTH_TOKEN=$(oc get secret model-token-service-account-token -n $PROJECT -o jsonpath='{.data.token}' | base64 --decode)
export ANTHROPIC_BASE_URL=https://$(oc get route qwen36-fp8 -n $PROJECT -o jsonpath='{.spec.host}')
export ANTHROPIC_MODEL="qwen36-fp8"
export ANTHROPIC_SMALL_FAST_MODEL="qwen36-fp8"
```


## Test model connectivity
```shell
curl -X POST "$ANTHROPIC_BASE_URL"/v1/chat/completions \
        -H "Content-Type: application/json" \
        -H "Authorization: Bearer ${ANTHROPIC_AUTH_TOKEN}" \
        -d '{
          "model": "qwen36-fp8",
          "messages": [
            {"role": "user", "content": "Hello, how are you?"}
          ],
          "max_tokens": 5
        }'
```


## Run Claude
```shell
claude
```

In claude, run `/model` and make sure `qwen36-fp8` is selected.