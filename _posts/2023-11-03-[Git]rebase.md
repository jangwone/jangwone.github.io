---
layout: post
title:  [Git]rebase
tags: git
---

<h2 id="gitrebase-정리">[Git]rebase 정리</h2>
<p>다음과 같이 branch가 있다고 생각해보자.</p>
<ul>
<li>환경설정완료</li>
<li>로그인 퇴근</li>
<li>로그인 아파서 퇴근</li>
<li>로그인 완료</li>
</ul>
<pre><code>$ git log
commit 35bd12b8487d517600c329620e3aa1eb0722ac1c (HEAD -&gt; master)
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:10:43 2023 +0900

    로그인완료

commit 37f79e8e8d783649dc2d8d6bd3db56d3f52f7175
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:10:28 2023 +0900

    아파서퇴근

commit 077f419d0316b30e242909ec1065fadd4dea0482
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:09:35 2023 +0900

    로그인퇴근

commit f5174fd8487f2916be7e9e2c9e75c8a4a2c3960c
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:09:07 2023 +0900

    환경설정완료
</code></pre>
<p>중간 부분에 “로그인퇴근”, “아파서퇴근” 커밋 로그를 정리를 하려고 한다.</p>
<pre><code>$ git rebase -i HEAD~3
</code></pre>
<p><code>rebase</code> 명령어를 사용하면 script가 열린다.</p>
<pre><code>pick 077f419 로그인퇴근
pick 37f79e8 아파서퇴근
pick 35bd12b 로그인완료
</code></pre>
<h2 id="git-rebase-drop-삭제">git rebase drop [삭제]</h2>
<p><code>drop</code> 옵션을 사용하여 commit 내역을 삭제해보자<br>
삭제하려는 부분의 옵션을 <code>d</code> 로 변경해주면 된다.</p>
<pre><code>pick 077f419 로그인퇴근
d 37f79e8 아파서퇴근
pick 35bd12b 로그인완료
</code></pre>
<p><code>git log</code>를 확인해보니 아파서퇴근 commit 내역이 삭제된 걸 확인할 수 있다.</p>
<pre><code>$ git log
commit 467d94d499c52fbf94f2c74e538ff74f55254c35 (HEAD -&gt; master)
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:10:43 2023 +0900

    로그인완료

commit 077f419d0316b30e242909ec1065fadd4dea0482
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:09:35 2023 +0900

    로그인퇴근

commit f5174fd8487f2916be7e9e2c9e75c8a4a2c3960c
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:09:07 2023 +0900

    환경설정완료
</code></pre>
<h2 id="git-reflog">git reflog</h2>
<p>하지만 이렇게 하면 작업 내역까지 삭제되어 drop 하면 안된다.<br>
<code>git reflog</code> 명령어를 통해 이전까지 작업했던 내용을 확인해보자</p>
<pre><code>$ git reflog
467d94d (HEAD -&gt; master) HEAD@{0}: rebase (finish): returning to refs/heads/master
467d94d (HEAD -&gt; master) HEAD@{1}: rebase (pick): 로그인완료
077f419 HEAD@{2}: rebase (start): checkout HEAD~3
35bd12b HEAD@{3}: commit: 로그인완료
37f79e8 HEAD@{4}: commit: 아파서퇴근
077f419 HEAD@{5}: commit: 로그인퇴근
f5174fd HEAD@{6}: commit (initial): 환경설정완료
</code></pre>
<h2 id="git-reset---hard">git reset --hard</h2>
<p>rebase 이전의 상태로 돌아가면 지워진 내용을 복구할 수 있다.<br>
<code>git reset --hard</code> 명령어를 사용하여 이전 상태로 복구하자</p>
<pre><code>$ git reset --hard 35bd
HEAD is now at 35bd12b 로그인완료

$ git log
commit 35bd12b8487d517600c329620e3aa1eb0722ac1c (HEAD -&gt; master)
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:10:43 2023 +0900

    로그인완료

commit 37f79e8e8d783649dc2d8d6bd3db56d3f52f7175
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:10:28 2023 +0900

    아파서퇴근

commit 077f419d0316b30e242909ec1065fadd4dea0482
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:09:35 2023 +0900

    로그인퇴근

commit f5174fd8487f2916be7e9e2c9e75c8a4a2c3960c
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:09:07 2023 +0900

    환경설정완료
</code></pre>
<p>다시 이전의 상태로 돌아왔다.</p>
<h2 id="git-rebase-reword-수정">git rebase reword [수정]</h2>
<p><code>rebase</code> 명령어로 script를 열어 변경 할 <code>commit log</code>옵션을 <code>r</code>로 지정한다.<br>
script를 저장하고 종료하면 <code>commit log</code>를 변경할 수 있다.<br>
<code>아파서퇴근</code>을 <code>아파서빨리퇴근</code>으로 변경했다.</p>
<pre><code>$ git rebase -i HEAD~3

pick 077f419 로그인퇴근
r 37f79e8 아파서퇴근
pick 35bd12b 로그인완료

# 아파서빨리퇴근으로 커밋내역 변경

$ git log
commit 381cdee3e11b59c3405b8c5941d50a3177323e58 (HEAD -&gt; master)
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:10:43 2023 +0900

    로그인완료

commit f812ce4f6d6a0fc64c6af242c4ab7d824df65582
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:10:28 2023 +0900

    아파서빨리퇴근

commit 077f419d0316b30e242909ec1065fadd4dea0482
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:09:35 2023 +0900

    로그인퇴근

commit f5174fd8487f2916be7e9e2c9e75c8a4a2c3960c
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:09:07 2023 +0900

    환경설정완료
</code></pre>
<h2 id="git-rebase-squash-정리">git rebase squash [정리]</h2>
<p><code>rebase</code>를 통해 확실하게 commit을 정리해보자</p>
<ul>
<li>로그인퇴근</li>
<li>아파서빨리퇴근</li>
<li>로그인완료</li>
</ul>
<p>3개의 커밋 내역을 로그인 1개로 바꾸려고 한다.</p>
<ul>
<li>로그인</li>
</ul>
<pre><code>$ git rebase -i HEAD~3

pick 077f419 로그인퇴근
s f812ce4 아파서빨리퇴근
s 381cdee 로그인완료
</code></pre>
<p>첫번째 commit 다음부터  <code>s</code> 옵션을 부여한다.<br>
저장하고 종료한다.</p>
<pre><code># This is a combination of 3 commits.
# This is the 1st commit message:

로그인퇴근

# This is the commit message #2:

아파서빨리퇴근

# This is the commit message #3:

로그인완료
</code></pre>
<p>아래와 같이 <code>마지막</code> commit 메시지만 작성한다.</p>
<pre><code># This is a combination of 3 commits.
# This is the 1st commit message:


# This is the commit message #2:



# This is the commit message #3:

로그인
</code></pre>
<p>git log를 통해 commit이 정리된 걸 확인할 수 있다.<br>
<code>s</code> 옵션을 사용하면 여러 개의 commit을 하나의 commit으로 묶어버릴 수 있다.</p>
<pre><code>$ git log
commit 86c94d1da4754da2149d5b0d00897f0c078b7c35 (HEAD -&gt; master)
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:09:35 2023 +0900

    로그인

commit f5174fd8487f2916be7e9e2c9e75c8a4a2c3960c
Author: jangwone &lt;87637341+jangwone@users.noreply.github.com&gt;
Date:   Fri Nov 3 16:09:07 2023 +0900

    환경설정완료
</code></pre>

