## Troubleshooting

### VS Code cannot import `dotenv`

If the notebook reports `ModuleNotFoundError: No module named 'dotenv'`, the notebook may be using a different Python interpreter from the lesson's virtual environment.

First, install the dependencies using the virtual environment's Python:

```bash
./lesson-01-agent-basics/.venv/bin/python -m pip install openai python-dotenv ipykernel
```

Then, in VS Code:

1. Open the Command Palette.
2. Select **Python: Select Interpreter**.
3. Choose **Enter interpreter path...**
4. Select `lesson-01-agent-basics/.venv/bin/python`.
5. Return to the notebook and select the same environment as its kernel.

To confirm which Python environment the notebook is using, run:

```python
import sys

print(sys.executable)
```

The displayed path should end with:

```text
lesson-01-agent-basics/.venv/bin/python
```

The `.env` file and installed packages belong to separate parts of the setup: `.env` stores local configuration, while the selected Python environment determines which packages the notebook can import.
