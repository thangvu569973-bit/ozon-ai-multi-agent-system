ozon-ai-system/
│
├── app.py                  # Web入口（Flask）
├── config.py               # 配置
├── requirements.txt
│
├── core/
│   ├── llm.py              # LLM封装
│   ├── orchestrator.py     # Agent调度器（核心）
│
├── agents/
│   ├── base.py             # Agent基类
│   ├── product_agent.py    # 选品
│   ├── listing_agent.py    # Listing
│   ├── ads_agent.py        # 广告策略
│   ├── analytics_agent.py  # 数据分析
│
├── services/
│   ├── ozon_client.py      # 未来对接Ozon API
│   ├── data_service.py     # 数据层
│
├── workflows/
│   ├── pipeline.py         # 工作流定义
│
├── templates/
│   └── index.html
│
└── static/
    └── style.css
    # core/llm.py

class LLM:
    def __init__(self, model="gpt-4o-mini"):
        self.model = model

    def call(self, prompt: str) -> str:
        # 这里未来可接 OpenAI / Claude / 本地模型
        return f"[LLM RESPONSE MOCK]\n{prompt[:200]}"
        # agents/base.py

class BaseAgent:
    def __init__(self, llm):
        self.llm = llm
        self.name = self.__class__.__name__

    def run(self, input_data: dict):
        raise NotImplementedError
        # agents/product_agent.py

from agents.base import BaseAgent

class ProductAgent(BaseAgent):

    def run(self, input_data):
        prompt = f"""
Analyze product for Ozon marketplace:

Product: {input_data}

Return:
- competition level
- profit estimate
- market opportunity
"""
        return self.llm.call(prompt)
  # agents/listing_agent.py

from agents.base import BaseAgent

class ListingAgent(BaseAgent):

    def run(self, input_data):
        prompt = f"""
Generate Ozon listing (Russian):

Product analysis:
{input_data}

Output:
- Title
- Bullet points
- Keywords
"""
        return self.llm.call(prompt)
  # agents/ads_agent.py

from agents.base import BaseAgent

class AdsAgent(BaseAgent):

    def run(self, input_data):
        prompt = f"""
Create advertising strategy for Ozon:

{input_data}

Include:
- budget allocation
- keyword bidding
- optimization strategy
"""
        return self.llm.call(prompt)
  # core/orchestrator.py

from agents.product_agent import ProductAgent
from agents.listing_agent import ListingAgent
from agents.ads_agent import AdsAgent

class Orchestrator:

    def __init__(self, llm):
        self.product_agent = ProductAgent(llm)
        self.listing_agent = ListingAgent(llm)
        self.ads_agent = AdsAgent(llm)

    def run_pipeline(self, product_input):

        # Step 1: 选品分析
        analysis = self.product_agent.run(product_input)

        # Step 2: Listing生成
        listing = self.listing_agent.run(analysis)

        # Step 3: 广告策略
        ads = self.ads_agent.run(analysis)

        return {
            "analysis": analysis,
            "listing": listing,
            "ads": ads
        }
        # app.py

from flask import Flask, render_template, request
from core.llm import LLM
from core.orchestrator import Orchestrator

app = Flask(__name__)

llm = LLM()
engine = Orchestrator(llm)

@app.route("/", methods=["GET", "POST"])
def index():
    result = None

    if request.method == "POST":
        product = request.form.get("product")
        result = engine.run_pipeline(product)

    return render_template("index.html", result=result)

if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=3000)
    <!DOCTYPE html>
<html>
<head>
    <title>Ozon AI System</title>
    <style>
        body { font-family: Arial; background:#0f172a; color:white; padding:40px; }
        input, button { padding:10px; width:300px; margin:5px; }
        button { background:#22c55e; color:white; border:none; }
        .box { background:#1e293b; padding:20px; margin-top:20px; }
    </style>
</head>
<body>

<h1>Ozon AI Multi-Agent System</h1>

<form method="post">
    <input name="product" placeholder="Enter product">
    <button type="submit">Run Pipeline</button>
</form>

{% if result %}
<div class="box">
    <h3>Analysis</h3>
    <pre>{{result.analysis}}</pre>

    <h3>Listing</h3>
    <pre>{{result.listing}}</pre>

    <h3>Ads</h3>
    <pre>{{result.ads}}</pre>
</div>
{% endif %}

</body>
</html>
flask
