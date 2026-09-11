<h2 data-section-id="1jts7sa" data-start="140" data-end="170">LLM Dead Drop Collaboration</h2>
<h3 data-section-id="3p1dlm" data-start="172" data-end="209">Tested with Codex, Claude Code and Gemini</h3>
<p data-start="211" data-end="344">A drop-in protocol for coordinating two LLM coding agents through a shared <code data-start="286" data-end="300">agentchat.md</code> file and zero-data dead-drop notifications. <br /><br />It is possible to add multiple LLMs together into a swarm using this communication method. With further instruction roles can be customised which opens up the possibility of things such as role assignment by various dynamic methods and other fun chaos.</p>
<h3 data-section-id="l6ho55" data-start="346" data-end="356">Method</h3>
<p data-start="358" data-end="575">Each agent checks its designated dead-drop file at the start and end of a work turn. The dead-drop file contains <strong data-start="471" data-end="489">no information</strong>: its existence only means <em data-start="516" data-end="575">&ldquo;re-read <code data-start="526" data-end="540">agentchat.md</code>; coordination state has changed.&rdquo;</em></p>
<p data-start="577" data-end="751">When an agent finds its notice, it claims it atomically, reads the newest relevant entries in <code data-start="671" data-end="685">agentchat.md</code>, responds as required, then signals the other agent if necessary.</p>
<p data-start="753" data-end="943">The agents use deliberately adversarial peer review rather than assuming agreement. Design, implementation and evidence can therefore be challenged before they become accepted project state.</p>
<h3 data-section-id="dx0ce6" data-start="945" data-end="977">Persistent failure reminders</h3>
<p data-start="979" data-end="1046">A useful additional instruction is given privately to the reviewer:</p>
<blockquote data-start="1048" data-end="1224">
<p data-start="1050" data-end="1224">If the other agent makes a major mistake, or repeatedly makes the same expensive mistake, keep bringing that failure class up until it is genuinely no longer a credible risk.</p>
</blockquote>
<p data-start="1226" data-end="1412">This creates a persistent two-sided reminder: one agent knows what it previously got wrong; the reviewer knows what to keep checking. It also becomes an informal running problem tracker.</p>
<h3 data-section-id="qs0ub6" data-start="1414" data-end="1432">Project memory</h3>
<p data-start="1434" data-end="1506">The collaboration system can be paired with two growing project manuals:</p>
<ul data-start="1508" data-end="1706">
<li data-section-id="xbreug" data-start="1508" data-end="1601"><strong data-start="1510" data-end="1529">Success manual:</strong> tested approaches, known-good mechanisms and established project facts.</li>
<li data-section-id="9ekjwq" data-start="1602" data-end="1706"><strong data-start="1604" data-end="1623">Failure manual:</strong> disproven assumptions, failed approaches, misleading symptoms and recurring traps.</li>
</ul>
<p data-start="1708" data-end="1939">External research can be added as indexed notes after its claims have been tested against the project. This reduces repeated investigation and gives both agents a shared body of evidence against which new designs can be challenged.</p>
<h3 data-section-id="1ptk521" data-start="1941" data-end="1955">Automation</h3>
<p data-start="1957" data-end="2165">Dead-drop checks can also be scheduled while the agents are otherwise idle. The important rule remains the same: the notice is only a zero-data wake-up flag; the actual message always lives in <code data-start="2150" data-end="2164">agentchat.md</code>.<br /><br /><strong>IMPORTANT NOTE</strong></p>
<p>Obviously use github, limit folder access, by downloading this you accept all responsibility if this rm -rf's everything.<br /><br /><br /><br /></p>
