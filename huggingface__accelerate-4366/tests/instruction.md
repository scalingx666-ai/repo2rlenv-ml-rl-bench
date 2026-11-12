# Issue

**Title:** chore(deps): bump the actions group with 4 updates

## Description

Bumps the actions group with 4 updates: [huggingface/doc-builder/.github/workflows/build_main_documentation.yml](https://github.com/huggingface/doc-builder), [huggingface/doc-builder/.github/workflows/build_pr_documentation.yml](https://github.com/huggingface/doc-builder), [trufflesecurity/trufflehog](https://github.com/trufflesecurity/trufflehog) and [huggingface/doc-builder/.github/workflows/upload_pr_documentation.yml](https://github.com/huggingface/doc-builder).

Updates `huggingface/doc-builder/.github/workflows/build_main_documentation.yml` from 63ec6f1a0ac9300e6a35f4efe140b09ac9f68a89 to 7e31a94f70994bd73656a81690b13aaed439e798
<details>
<summary>Changelog</summary>
<p><em>Sourced from <a href="https://github.com/huggingface/doc-builder/blob/main/RELEASE.md">huggingface/doc-builder/.github/workflows/build_main_documentation.yml's changelog</a>.</em></p>
<blockquote>
<h1>Release checklist</h1>
<ol>
<li>Checkout the release branch (for a patch the current release branch, for a new minor version, create one):
<pre lang="bash"><code>git checkout -b vXX.xx-release
</code></pre>
The <code>-b</code> is only necessary for creation (so remove it when doing a patch).</li>
<li>Change the version in <code>src/doc_builder/__init__.py</code> and <code>pyproject.toml</code> to the proper value.</li>
<li>Commit these changes with the message: <code>&quot;Release: v&lt;VERSION&gt;&quot;</code>.</li>
<li>Add a tag in git to mark the release:
<pre lang="bash"><code>git tag v&lt;VERSION&gt; -m 'Adds tag v&lt;VERSION&gt; for pypi'
</code></pre>
Push the tag and release commit to git:
<pre lang="bash"><code>git push --tags origin vXX.xx-release
</code></pre>
</li>
<li>Build the source distribution and the wheel in the top-level directory:
<pre lang="bash"><code>rm -rf dist
uv build
</code></pre>
</li>
<li>Upload the package to the pypi test server first:
<pre lang="bash"><code>twine upload dist/* -r testpypi
</code></pre>
</li>
<li>Check that you can install it in a virtualenv by running:
<pre lang="bash"><code>pip install hf-doc-builder
pip uninstall hf-doc-builder
pip install -i https://test.pypi.org/simple/ hf-doc-builder
</code></pre>
It's recommended to check that there are no issues building the docs, so try running a command like <code>doc-builder</code>.</li>
<li>Upload the final version to actual pypi:
<pre lang="bash"><code>twine upload dist/* -r pypi
</code></pre>
</li>
<li>Add release notes to the tag in github once everything is looking hunky-dory.</li>
<li>Go back to the main branch and update the version in <code>src/doc_builder/__init__.py</code> and <code>pyproject.toml</code> to the new
version &quot;.dev&quot; and push to main.</li>
</ol>
</blockquote>
</details>
<details>
<summary>Commits</summary>
<ul>
<li><a href=""><code>7e31a94</code></a> [meilisearch] keep the incremental-update tracker in sync with the index (<a href="">#845</a>)</li>
<li><a href=""><code>bc117cf</code></a> [meilisearch] use the rendered heading anchors in search result URLs (<a href="">#844</a>)</li>
<li><a href=""><code>9d137cb</code></a> Mock transformers' optional audio backends so autodoc stops documenting place...</li>
<li><a href=""><code>db72037</code></a> [meilisearch] add plain-text <code>text_plain</code> field and clean headings (<a href="">#843</a>)</li>
<li><a href=""><code>61704e2</code></a> Merge commit from fork</li>
<li><a href=""><code>ef23fb8</code></a> [meilisearch] make <code>id</code> filterable in docs-semantic-search (<a href="">#841</a>)</li>
<li>See full diff in <a href="https://github.com/huggingface/doc-builder/compare/63ec6f1a0ac9300e6a35f4efe140b09ac9f68a89...7e31a94f70994bd73656a81690b13aaed439e798">compare view</a></li>
</ul>
</details>
<br />

Updates `huggingface/doc-builder/.github/workflows/build_pr_documentation.yml` from 63ec6f1a0ac9300e6a35f4efe140b09ac9f68a89 to 7e31a94f70994bd73656a81690b13aaed439e798
<details>
<summary>Changelog</summary>
<p><em>Sourced from <a href="https://github.com/huggingface/doc-builder/blob/main/RELEASE.md">huggingface/doc-builder/.github/workflows/build_pr_documentation.yml's changelog</a>.</em></p>
<blockquote>
<h1>Release checklist</h1>
<ol>
<li>Checkout the release branch (for a patch the current release branch, for a new minor version, create one):
<pre lang="bash"><code>git checkout -b vXX.xx-release
</code></pre>
The <code>-b</code> is only necessary for creation (so remove it when doing a patch).</li>
<li>Change the version in <code>src/doc_builder/__init__.py</code> and <code>pyproject.toml</code> to the proper value.</li>
<li>Commit these changes with the message: <code>&quot;Release: v&lt;VERSION&gt;&quot;</code>.</li>
<li>Add a tag in git to mark the release:
<pre lang="bash"><code>git tag v&lt;VERSION&gt; -m 'Adds tag v&lt;VERSION&gt; for pypi'
</code></pre>
Push the tag and release commit to git:
<pre lang="bash"><code>git push --tags origin vXX.xx-release
</code></pre>
</li>
<li>Build the source distribution and the wheel in the top-level directory:
<pre lang="bash"><code>rm -rf dist
uv build
</code></pre>
</li>
<li>Upload the package to the pypi test server first:
<pre lang="bash"><code>twine upload dist/* -r testpypi
</code></pre>
</li>
<li>Check that you can install it in a virtualenv by running:
<pre lang="bash"><code>pip install hf-doc-builder
pip uninstall hf-doc-builder
pip install -i https://test.pypi.org/simple/ hf-doc-builder
</code></pre>
It's recommended to check that there are no issues building the docs, so try running a command like <code>doc-builder</code>.</li>
<li>Upload the final version to actual pypi:
<pre lang="bash"><code>twine upload dist/* -r pypi
</code></pre>
</li>
<li>Add release notes to the tag in github once everything is looking hunky-dory.</li>
<li>Go back to the main branch and update the version in <code>src/doc_builder/__init__.py</code> and <code>pyproject.toml</code> to the new
version &quot;.dev&quot; and push to main.</li>
</ol>
</blockquote>
</details>
<details>
<summary>Commits</summary>
<ul>
<li><a href=""><code>7e31a94</code></a> [meilisearch] keep the incremental-update tracker in sync with the index (<a href="">#845</a>)</li>
<li><a href=""><code>bc117cf</code></a> [meilisearch] use the rendered heading anchors in search result URLs (<a href="">#844</a>)</li>
<li><a href=""><code>9d137cb</code></a> Mock transformers' optional audio backends so autodoc stops documenting place...</li>
<li><a href=""><code>db72037</code></a> [meilisearch] add plain-text <code>text_plain</code> field and clean headings (<a href="">#843</a>)</li>
<li><a href=""><code>61704e2</code></a> Merge commit from fork</li>
<li><a href=""><code>ef23fb8</code></a> [meilisearch] make <code>id</code> filterable in docs-semantic-search (<a href="">#841</a>)</li>
<li>See full diff in <a href="https://github.com/huggingface/doc-builder/compare/63ec6f1a0ac9300e6a35f4efe140b09ac9f68a89...7e31a94f70994bd73656a81690b13aaed439e798">compare view</a></li>
</ul>
</details>
<br />

Updates `trufflesecurity/trufflehog` from 3.97.5 to 3.97.9
<details>
<summary>Release notes</summary>
<p><em>Sourced from <a href="https://github.com/trufflesecurity/trufflehog/releases">trufflesecurity/trufflehog's releases</a>.</em></p>
<blockquote>
<h2>v3.97.9</h2>
<h2>What's Changed</h2>
<ul>
<li>[SCAN-162] Add Err() to JobProgress and JobProgressRef by <a href="https://github.com/dipto-truffle"><code>@​dipto-truffle</code></a> in <a href="">trufflesecurity/trufflehog#5346</a></li>
<li>ci: scope Smoke timeouts to trufflehog runs, not the build by <a href="https://github.com/bryanbeverly"><code>@​bryanbeverly</code></a> in <a href="">trufflesecurity/trufflehog#5317</a></li>
<li>Update module github.com/aymanbagabas/go-osc52 to v1.2.2 by <a href="https://github.com/renovate"><code>@​renovate</code></a>[bot] in <a href="">trufflesecurity/trufflehog#5252</a></li>
<li>Update module github.com/gabriel-vasile/mimetype to v1.4.15 by <a href="https://github.com/renovate"><code>@​renovate</code></a>[bot] in <a href="">trufflesecurity/trufflehog#5283</a></li>
<li>make 401s for Basic auth verified false. by <a href="https://github.com/jordanTunstill"><code>@​jordanTunstill</code></a> in <a href="">trufflesecurity/trufflehog#5290</a></li>
<li>[INT-1088] Add version and comment_number lines to SharePoint source metadata by <a href="https://github.com/camgunz"><code>@​camgunz</code></a> in <a href="">trufflesecurity/trufflehog#5348</a></li>
<li>Int 595 auth errors by <a href="https://github.com/shahzadhaider1"><code>@​shahzadhaider1</code></a> in <a href="">trufflesecurity/trufflehog#5259</a></li>
<li>Spruce up Makefile a little by <a href="https://github.com/camgunz"><code>@​camgunz</code></a> in <a href="">trufflesecurity/trufflehog#5347</a></li>
</ul>
<p><strong>Full Changelog</strong>: <a href="https://github.com/trufflesecurity/trufflehog/compare/v3.97.8...v3.97.9">https://github.com/trufflesecurity/trufflehog/compare/v3.97.8...v3.97.9</a></p>
<h2>v3.97.8</h2>
<h2>What's Changed</h2>
<ul>
<li>Add Okta OAuth support in Okta detector by <a href="https://github.com/shreyas-sriram"><code>@​shreyas-sriram</code></a> in <a href="">trufflesecurity/trufflehog#5323</a></li>
<li>Add per detector verification timing to verification cache by <a href="https://github.com/kvnphvm"><code>@​kvnphvm</code></a> in <a href="">trufflesecurity/trufflehog#5341</a></li>
<li>Set all verification errors in detectors by <a href="https://github.com/shahzadhaider1"><code>@​shahzadhaider1</code></a> in <a href="">trufflesecurity/trufflehog#5253</a></li>
</ul>
<p><strong>Full Changelog</strong>: <a href="https://github.com/trufflesecurity/trufflehog/compare/v3.97.7...v3.97.8">https://github.com/trufflesecurity/trufflehog/compare/v3.97.7...v3.97.8</a></p>
<h2>v3.97.7</h2>
<h2>What's Changed</h2>
<ul>
<li>Recognize a missing Postgres database by SQLSTATE by <a href="https://github.com/gaborbernat"><code>@​gaborbernat</code></a> in <a href="">trufflesecurity/trufflehog#5312</a></li>
<li>Improve Github App installation access token caching by <a href="https://github.com/mariduv"><code>@​mariduv</code></a> in <a href="">trufflesecurity/trufflehog#5324</a></li>
<li>give git the real base..head range for diff scans by <a href="https://github.com/kvnphvm"><code>@​kvnphvm</code></a> in <a href="">trufflesecurity/trufflehog#5325</a></li>
<li>Add GCS source documentation by <a href="https://github.com/kashifkhan0771"><code>@​kashifkhan0771</code></a> in <a href="">trufflesecurity/trufflehog#5319</a></li>
<li>[PERF] Remove \b anchor from escaped-unicode codePointPat by <a href="https://github.com/Ahsan-Sarbaz"><code>@​Ahsan-Sarbaz</code></a> in <a href="">trufflesecurity/trufflehog#5320</a></li>
<li>🐛 fix(engine): map decoded results to source lines by <a href="https://github.com/gaborbernat"><code>@​gaborbernat</code></a> in <a href="">trufflesecurity/trufflehog#5219</a></li>
</ul>
<h2>New Contributors</h2>
<ul>
<li><a href="https://github.com/gaborbernat"><code>@​gaborbernat</code></a> made their first contribution in <a href="">trufflesecurity/trufflehog#5312</a></li>
</ul>
<p><strong>Full Changelog</strong>: <a href="https://github.com/trufflesecurity/trufflehog/compare/v3.97.6...v3.97.7">https://github.com/trufflesecurity/trufflehog/compare/v3.97.6...v3.97.7</a></p>
<h2>v3.97.6</h2>
<h2>What's Changed</h2>
<ul>
<li>Clarify security policy by <a href="https://github.com/bradlarsen"><code>@​bradlarsen</code></a> in <a href="">trufflesecurity/trufflehog#5295</a></li>
<li>Add git source documentation by <a href="https://github.com/kashifkhan0771"><code>@​kashifkhan0771</code></a> in <a href="">trufflesecurity/trufflehog#5326</a></li>
<li>fix(gitparse): cheaper low-memory git scan, stop losing trailing commits by <a href="https://github.com/kashifkhan0771"><code>@​kashifkhan0771</code></a> in <a href="">trufflesecurity/trufflehog#5331</a></li>
<li>analyzer/github: include is_fine_grained metadata and make gist enumeration non-fatal by <a href="https://github.com/unsmith"><code>@​unsmith</code></a> in <a href="">trufflesecurity/trufflehog#5332</a></li>
</ul>
<p><strong>Full Changelog</strong>: <a href="https://github.com/trufflesecurity/trufflehog/compare/v3.97.5...v3.97.6">https://github.com/trufflesecurity/trufflehog/compare/v3.97.5...v3.97.6</a></p>
</blockquote>
</details>
<details>
<summary>Commits</summary>
<ul>
<li><a href=""><code>4dd8831</code></a> Spruce up Makefile a little (<a href="">#5347</a>)</li>
<li><a href=""><code>449d8a3</code></a> Int 595 auth errors (<a href="">#5259</a>)</li>
<li><a href=""><code>bad9901</code></a> Add version and comment_number lines to SharePoint source metadata (<a href="">#5348</a>)</li>
<li><a href=""><code>4b8eb0e</code></a> make 401s for Basic auth verified false. (<a href="">#5290</a>)</li>
<li><a href=""><code>bbf9447</code></a> Update module github.com/gabriel-vasile/mimetype to v1.4.15 (<a href="">#5283</a>)</li>
<li><a href=""><code>16b566b</code></a> Update module github.com/aymanbagabas/go-osc52 to v1.2.2 (<a href="">#5252</a>)</li>
<li><a href=""><code>a25ff85</code></a> ci: scope Smoke timeouts to trufflehog runs, not the build (<a href="">#5317</a>)</li>
<li><a href=""><code>ca9d3b3</code></a> [SCAN-162] Add Err() to JobProgress and JobProgressRef (<a href="">#5346</a>)</li>
<li><a href=""><code>a5f3de5</code></a> Set all verification errors in detectors (<a href="">#5253</a>)</li>
<li><a href=""><code>7ee4d49</code></a> Add per detector verification timing to verification cache (<a href="">#5341</a>)</li>
<li>Additional commits viewable in <a href="https://github.com/trufflesecurity/trufflehog/compare/f714bf454f350590f4a24c3ddb1aef02c35bf5b6...4dd8831c5f12599465d4d45c3c447b4018a34c85">compare view</a></li>
</ul>
</details>
<br />

Updates `huggingface/doc-builder/.github/workflows/upload_pr_documentation.yml` from 63ec6f1a0ac9300e6a35f4efe140b09ac9f68a89 to 7e31a94f70994bd73656a81690b13aaed439e798
<details>
<summary>Changelog</summary>
<p><em>Sourced from <a href="https://github.com/huggingface/doc-builder/blob/main/RELEASE.md">huggingface/doc-builder/.github/workflows/upload_pr_documentation.yml's changelog</a>.</em></p>
<blockquote>
<h1>Release checklist</h1>
<ol>
<li>Checkout the release branch (for a patch the current release branch, for a new minor version, create one):
<pre lang="bash"><code>git checkout -b vXX.xx-release
</code></pre>
The <code>-b</code> is only necessary for creation (so remove it when doing a patch).</li>
<li>Change the version in <code>src/doc_builder/__init__.py</code> and <code>pyproject.toml</code> to the proper value.</li>
<li>Commit these changes with the message: <code>&quot;Release: v&lt;VERSION&gt;&quot;</code>.</li>
<li>Add a tag in git to mark the release:
<pre lang="bash"><code>git tag v&lt;VERSION&gt; -m 'Adds tag v&lt;VERSION&gt; for pypi'
</code></pre>
Push the tag and release commit to git:
<pre lang="bash"><code>git push --tags origin vXX.xx-release
</code></pre>
</li>
<li>Build the source distribution and the wheel in the top-level directory:
<pre lang="bash"><code>rm -rf dist
uv build
</code></pre>
</li>
<li>Upload the package to the pypi test server first:
<pre lang="bash"><code>twine upload dist/* -r testpypi
</code></pre>
</li>
<li>Check that you can install it in a virtualenv by running:
<pre lang="bash"><code>pip install hf-doc-builder
pip uninstall hf-doc-builder
pip install -i https://test.pypi.org/simple/ hf-doc-builder
</code></pre>
It's recommended to check that there are no issues building the docs, so try running a command like <code>doc-builder</code>.</li>
<li>Upload the final version to actual pypi:
<pre lang="bash"><code>twine upload dist/* -r pypi
</code></pre>
</li>
<li>Add release notes to the tag in github once everything is looking hunky-dory.</li>
<li>Go back to the main branch and update the version in <code>src/doc_builder/__init__.py</code> and <code>pyproject.toml</code> to the new
version &quot;.dev&quot; and push to main.</li>
</ol>
</blockquote>
</details>
<details>
<summary>Commits</summary>
<ul>
<li><a href=""><code>7e31a94</code></a> [meilisearch] keep the incremental-update tracker in sync with the index (<a href="">#845</a>)</li>
<li><a href=""><code>bc117cf</code></a> [meilisearch] use the rendered heading anchors in search result URLs (<a href="">#844</a>)</li>
<li><a href=""><code>9d137cb</code></a> Mock transformers' optional audio backends so autodoc stops documenting place...</li>
<li><a href=""><code>db72037</code></a> [meilisearch] add plain-text <code>text_plain</code> field and clean headings (<a href="">#843</a>)</li>
<li><a href=""><code>61704e2</code></a> Merge commit from fork</li>
<li><a href=""><code>ef23fb8</code></a> [meilisearch] make <code>id</code> filterable in docs-semantic-search (<a href="">#841</a>)</li>
<li>See full diff in <a href="https://github.com/huggingface/doc-builder/compare/63ec6f1a0ac9300e6a35f4efe140b09ac9f68a89...7e31a94f70994bd73656a81690b13aaed439e798">compare view</a></li>
</ul>
</details>
<br />

Dependabot will resolve any conflicts with this PR as long as you don't alter it yourself. You can also trigger a rebase manually by commenting `@dependabot rebase`.

[//]: # (dependabot-automerge-start)
[//]: # (dependabot-automerge-end)

---

<details>
<summary>Dependabot commands and options</summary>
<br />

You can trigger Dependabot actions by commenting on this PR:
- `@dependabot rebase` will rebase this PR
- `@dependabot recreate` will recreate this PR, overwriting any edits that have been made to it
- `@dependabot show <dependency name> ignore conditions` will show all of the ignore conditions of the specified dependency
- `@dependabot ignore <dependency name> major version` will close this group update PR and stop Dependabot creating any more for the specific dependency's major version (unless you unignore this specific dependency's major version or upgrade to it yourself)
- `@dependabot ignore <dependency name> minor version` will close this group update PR and stop Dependabot creating any more for the specific dependency's minor version (unless you unignore this specific dependency's minor version or upgrade to it yourself)
- `@dependabot ignore <dependency name>` will close this group update PR and stop Dependabot creating any more for the specific dependency (unless you unignore this specific dependency or upgrade to it yourself)
- `@dependabot unignore <dependency name>` will remove all of the ignore conditions of the specified dependency
- `@dependabot unignore <dependency name> <ignore condition>` will remove the ignore condition of the specified dependency and ignore conditions

</details>

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `01c73fbdb9a7`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.