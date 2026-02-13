# Amazon Bedrock - Kimi K2.5 Reasoning Configuration Test

A Python script to demonstrate and validate how to enable the thinking/reasoning feature for the Kimi K2.5 model (moonshotai.kimi-k2.5) on Amazon Bedrock.

## Background

The Kimi K2.5 model supports a thinking/reasoning mode where the model shows its chain-of-thought reasoning process before providing a final answer. However, the parameter format used on Amazon Bedrock differs from the Moonshot native API.

On the Moonshot native API, reasoning is controlled via:
json
"thinking": {"type": "enabled"}
plaintext
Copy
On Amazon Bedrock, this parameter is silently ignored. Instead, you need to use the following in `additionalModelRequestFields`:
json
"additionalModelRequestFields": {
    "reasoning_config": "high"
}
plaintext
Copy
## Findings

| Configuration | Reasoning Content Returned? | Notes |
|---|---|---|
| `"thinking": {"type": "enabled"}` | No | Moonshot native format, silently ignored on Bedrock |
| `"reasoning_config": "high"` | Yes | Correct Bedrock format, reasoning content returned |
| No config (default) | No | Default behaviour, no reasoning |

## Prerequisites

- Python 3.8+
- AWS credentials configured with access to Amazon Bedrock
- Access to the Kimi K2.5 model (moonshotai.kimi-k2.5) in your region
- boto3 installed
bash
pip install boto3
plaintext
Copy
## Usage
bash
python k2.5-replication.py
plaintext
Copy
By default the script targets `us-east-1`. To change the region, modify the `region_name` parameter in the script.

## Regional Availability

As of February 2026, Kimi K2.5 is not available in all regions. The script has been tested and confirmed working in `us-east-1`. Check the [Amazon Bedrock model support page](https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html) for the latest regional availability.

## Example Output

The script runs three tests and produces output similar to the following (truncated for brevity):

============================================================
TEST: thinking = {'type': 'enabled'} (Moonshot native format)
============================================================
Stop reason: end_turn
Usage: {
  "inputTokens": 43,
  "outputTokens": 86,
  "totalTokens": 129
}

[TEXT]: I need to calculate 25 * 47 + 133 step by step...

[NO REASONING CONTENT RETURNED]

============================================================
TEST: reasoning_config = 'high'
============================================================
Stop reason: end_turn
Usage: {
  "inputTokens": 42,
  "outputTokens": 312,
  "totalTokens": 354
}

[REASONING]: The user wants me to solve the math problem: 25 * 47 + 133...

[TEXT]: Here's the step-by-step calculation...

============================================================
TEST: No reasoning_config (default)
============================================================
Stop reason: end_turn
Usage: {
  "inputTokens": 43,
  "outputTokens": 95,
  "totalTokens": 138
}

[TEXT]: I need to solve 25 * 47 + 133 step by step...

[NO REASONING CONTENT RETURNED]
plaintext
Copy
## Quick Reference

To enable reasoning for Kimi K2.5 using the Bedrock Converse API:
python
import boto3

bedrock_runtime = boto3.client(
    service_name='bedrock-runtime',
    region_name='us-east-1'
)

response = bedrock_runtime.converse(
    modelId='moonshotai.kimi-k2.5',
    messages=[
        {
            'role': 'user',
            'content': [
                {'text': 'Your prompt here'}
            ]
        }
    ],
    inferenceConfig={
        'maxTokens': 8192,
        'temperature': 1.0
    },
    additionalModelRequestFields={
        'reasoning_config': 'high'
    }
)
plaintext
Copy
## References

- [Moonshot Kimi K2.5 Documentation](https://platform.moonshot.ai/docs/guide/kimi-k2-5-quickstart)
- [Amazon Bedrock Kimi K2.5 Launch Announcement](https://aws.amazon.com/about-aws/whats-new/2026/02/amazon-bedrock-adds-support-six-open-weights-models/)
- [Amazon Bedrock Reasoning Configuration Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/kb-test-configure-reasoning.html)

## License

This project is provided as-is for educational and troubleshooting purposes.
