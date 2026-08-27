2026-08-27 10:46

Status:

Tags: [[D5]] [[DP13]] [[Github]]


# Asana auto sync


name: Asana Sync

on:
  schedule:
    - cron: '*/15 * * * *'  
  workflow_dispatch:        

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Run Sync Script
        env:
          ASANA_TOKEN: ${{ secrets.ASANA_TOKEN }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          PROJECT_GID: ${{ secrets.PROJECT_GID }}
        run: python asana_sync.py



### References
[[Streamlit Div 5 Ticket App]]