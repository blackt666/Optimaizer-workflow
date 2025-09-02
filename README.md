# Optimaizer-workflow
name: Mirror import from -newsletter-manager-
on:
  workflow_dispatch:

permissions:
  contents: write

jobs:
  mirror:
    runs-on: ubuntu-latest
    steps:
      - name: Configure Git
        run: |
          git config --global user.name "github-actions[bot]"
          git config --global user.email "41898282+github-actions[bot]@users.noreply.github.com"

      - name: Install Git LFS
        run: |
          sudo apt-get update
          sudo apt-get install -y git-lfs
          git lfs install

      - name: Mirror clone private source (bare) over HTTPS with PAT
        env:
          SOURCE_REPO_PAT: ${{ secrets.SOURCE_REPO_PAT }}
        run: |
          git clone --mirror "https://x-access-token:${SOURCE_REPO_PAT}@github.com/blackt666/-newsletter-manager-.git" source.git

      - name: Push mirror to this repository
        env:
          TARGET_REPO: ${{ github.repository }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          cd source.git
          git remote remove origin || true
          git remote add origin "https://x-access-token:${GITHUB_TOKEN}@github.com/${TARGET_REPO}.git"
          git push --mirror

      - name: Transfer all LFS objects
        env:
          SOURCE_REPO_PAT: ${{ secrets.SOURCE_REPO_PAT }}
          TARGET_REPO: ${{ github.repository }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          git clone --no-tags --filter=blob:none "https://x-access-token:${SOURCE_REPO_PAT}@github.com/blackt666/-newsletter-manager-.git" work
          cd work
          git lfs fetch --all
          git remote remove origin || true
          git remote add origin "https://x-access-token:${GITHUB_TOKEN}@github.com/${TARGET_REPO}.git"
          git lfs push --all origin
