---
layout: default
title: Publications
permalink: /publications/
---

<div class="lang-ko">

<h1>논문</h1>

<div class="pub-item" data-paper-id="ArXiv:2509.21865">
  <div class="pub-title"><a href="https://arxiv.org/abs/2509.21865" target="_blank">Beyond RAG vs. Long-Context: Learning Distraction-Aware Retrieval for Efficient Knowledge Grounding</a></div>
  <div class="pub-authors">Seong-Woong Shim, Myunsoo Kim, <strong>Jae Hyeon Cho</strong>, Byungjun Lee</div>
  <div class="pub-venue">ICLR (The International Conference on Learning Representations), 2026 · 인용: <span class="cite-count">–</span></div>
</div>

<div class="pub-item" data-paper-id="ArXiv:2506.13513">
  <div class="pub-title"><a href="https://aclanthology.org/2025.acl-long.1039/" target="_blank">K/DA: Automated Data Generation Pipeline for Detoxifying Implicitly Offensive Language in Korean</a></div>
  <div class="pub-authors">Minkyeong Jeon*, Hyemin Jeong*, Yerang Kim, Jiyoung Kim, <strong>Jae Hyeon Cho</strong>, Byung-Jun Lee</div>
  <div class="pub-venue">ACL (The 63rd Annual Meeting of the Association for Computational Linguistics), 2025 · 인용: <span class="cite-count">–</span> · <a href="https://arxiv.org/abs/2506.13513" target="_blank">arXiv</a></div>
</div>

<div class="pub-item" data-paper-id="ArXiv:2506.12725">
  <div class="pub-title"><a href="https://aclanthology.org/2025.findings-emnlp.433/" target="_blank">Rethinking DPO: The Role of Rejected Responses in Preference Misalignment</a></div>
  <div class="pub-authors"><strong>Jae Hyeon Cho</strong>, JunHyeok Oh, Myunsoo Kim, Byung-Jun Lee</div>
  <div class="pub-venue">Findings of EMNLP (Empirical Methods in Natural Language Processing), 2025 · 인용: <span class="cite-count">–</span> · <a href="https://arxiv.org/abs/2506.12725" target="_blank">arXiv</a></div>
</div>

<div class="pub-item" data-paper-id="ArXiv:2410.22891">
  <div class="pub-title"><a href="https://direct.mit.edu/coli/article/doi/10.1162/COLI.a.579/133943/VPO-Leveraging-the-Number-of-Votes-in-Preference" target="_blank">VPO: Leveraging the Number of Votes in Preference Optimization</a></div>
  <div class="pub-authors"><strong>Jae Hyeon Cho</strong>, Minkyung Park, Byungjun Lee</div>
  <div class="pub-venue">Computational Linguistics, 2025 · 인용: <span class="cite-count">–</span> · <a href="https://arxiv.org/abs/2410.22891" target="_blank">arXiv</a></div>
</div>

</div>

<div class="lang-en">

<h1>Publications</h1>

<div class="pub-item" data-paper-id="ArXiv:2509.21865">
  <div class="pub-title"><a href="https://arxiv.org/abs/2509.21865" target="_blank">Beyond RAG vs. Long-Context: Learning Distraction-Aware Retrieval for Efficient Knowledge Grounding</a></div>
  <div class="pub-authors">Seong-Woong Shim, Myunsoo Kim, <strong>Jae Hyeon Cho</strong>, Byungjun Lee</div>
  <div class="pub-venue">ICLR (The International Conference on Learning Representations), 2026 · Citation: <span class="cite-count">–</span></div>
</div>

<div class="pub-item" data-paper-id="ArXiv:2506.13513">
  <div class="pub-title"><a href="https://aclanthology.org/2025.acl-long.1039/" target="_blank">K/DA: Automated Data Generation Pipeline for Detoxifying Implicitly Offensive Language in Korean</a></div>
  <div class="pub-authors">Minkyeong Jeon*, Hyemin Jeong*, Yerang Kim, Jiyoung Kim, <strong>Jae Hyeon Cho</strong>, Byung-Jun Lee</div>
  <div class="pub-venue">ACL (The 63rd Annual Meeting of the Association for Computational Linguistics), 2025 · Citation: <span class="cite-count">–</span> · <a href="https://arxiv.org/abs/2506.13513" target="_blank">arXiv</a></div>
</div>

<div class="pub-item" data-paper-id="ArXiv:2506.12725">
  <div class="pub-title"><a href="https://aclanthology.org/2025.findings-emnlp.433/" target="_blank">Rethinking DPO: The Role of Rejected Responses in Preference Misalignment</a></div>
  <div class="pub-authors"><strong>Jae Hyeon Cho</strong>, JunHyeok Oh, Myunsoo Kim, Byung-Jun Lee</div>
  <div class="pub-venue">Findings of EMNLP (Empirical Methods in Natural Language Processing), 2025 · Citation: <span class="cite-count">–</span> · <a href="https://arxiv.org/abs/2506.12725" target="_blank">arXiv</a></div>
</div>

<div class="pub-item" data-paper-id="ArXiv:2410.22891">
  <div class="pub-title"><a href="https://direct.mit.edu/coli/article/doi/10.1162/COLI.a.579/133943/VPO-Leveraging-the-Number-of-Votes-in-Preference" target="_blank">VPO: Leveraging the Number of Votes in Preference Optimization</a></div>
  <div class="pub-authors"><strong>Jae Hyeon Cho</strong>, Minkyung Park, Byungjun Lee</div>
  <div class="pub-venue">Computational Linguistics, 2025 · Citation: <span class="cite-count">–</span> · <a href="https://arxiv.org/abs/2410.22891" target="_blank">arXiv</a></div>
</div>

</div>

<script>
(function() {
  var items = document.querySelectorAll('.pub-item[data-paper-id]');
  var ids = [];
  items.forEach(function(el) {
    var id = el.getAttribute('data-paper-id');
    if (ids.indexOf(id) === -1) ids.push(id);
  });

  ids.forEach(function(id) {
    fetch('https://api.semanticscholar.org/graph/v1/paper/' + id + '?fields=citationCount')
      .then(function(r) { return r.json(); })
      .then(function(data) {
        if (data.citationCount !== undefined) {
          document.querySelectorAll('.pub-item[data-paper-id="' + id + '"] .cite-count').forEach(function(span) {
            span.textContent = data.citationCount;
          });
        }
      })
      .catch(function() {});
  });
})();
</script>
