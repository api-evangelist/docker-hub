---
title: "Gemma 4 is Here: Now Available on Docker Hub"
url: "https://www.docker.com/blog/gemma4-dockerhub/"
date: "Thu, 02 Apr 2026 16:16:56 +0000"
author: "Jennifer Angeles"
feed_url: "https://www.docker.com/feed/"
---
<p>Docker Hub is quickly becoming the home for AI models, serving millions of developers and bringing together a curated lineup that spans lightweight edge models to high-performance LLMs, all packaged as OCI artifacts.</p>



<p>Today, we’re excited to welcome <strong>Gemma 4</strong>, the latest generation of lightweight, state-of-the-art open models. Built on the same technology behind Gemini, Gemma 4 introduces three architectures that scale from low-power efficiency to high-end server performance.</p>



<p>By packaging models as OCI artifacts, models behave just like containers. They become versioned, shareable, and instantly deployable, with no custom toolchains required. You can pull ready-to-run models from Docker Hub, push your own, integrate with any OCI registry, and plug everything directly into your existing CI/CD pipelines using familiar tooling for security, access control, and automation.</p>



<p>And this is just the start. Over the next few weeks, Gemma 4 support is coming to Docker Model Runner, so you will not just discover models on Hub, you will be able to run, manage, and deploy them directly from Docker Desktop with the same simplicity you expect from Docker.</p>



<p>Docker Hub’s growing GenAI catalog already includes popular models like IBM Granite, Llama, Mistral, Phi, and SolarLLM, alongside apps like JupyterHub and H2O.ai, plus essential tools for inference, optimization, and orchestration.</p>



<h2 class="wp-block-heading"><strong><strong>What Docker Brings to Gemma 4</strong></strong></h2>



<p>Gemma 4 expands what efficient, high-performance models can do. Docker makes them simple to run, share, and scale anywhere.</p>



<ul class="wp-block-list">
<li><strong>Run efficiently at the edge</strong>: Smaller Gemma 4 variants are optimized for on-device performance. Docker enables consistent deployment across laptops, edge devices, and local environments.</li>



<li><strong>Scale performance with ease</strong>: From sparse to dense architectures, you can run any model like a container, making it easy to scale across cloud or on-prem infrastructure. </li>



<li><strong>One command to get started</strong>: Gemma 4 is just one command away:</li>
</ul>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: plain; title: ; notranslate">
docker model pull gemma4
</pre></div>


<p>No proprietary download tools. No custom authentication flows. Just the same pull, tag, push, and deploy workflow you already use.</p>



<p>By bringing Gemma 4 to Docker Hub, you get powerful models with a familiar, production-ready workflow.</p>



<h2 class="wp-block-heading"><strong>What’s New in Gemma 4?</strong></h2>



<p>Gemma 4 redefines what “small” models can do, with architectures optimized across multiple sizes and use cases:</p>



<ul class="wp-block-list">
<li><strong>Small &amp; Efficient (E2B, E4B):</strong> Built for on-device performance with high throughput and low memory use.</li>



<li><strong>Sparsely Activated (26B A4B): </strong>Mixture-of-Experts design delivers large-model quality with smaller-model speed.</li>



<li><strong>Flagship Dense (31B):</strong> High-performance model with a 256K context window for long-context reasoning.</li>
</ul>



<p>Key capabilities include multimodal support (text, image, audio), advanced reasoning with “thinking” tokens, and strong coding plus function-calling abilities.</p>



<h2 class="wp-block-heading"><strong>Technical Specifications</strong></h2>



<p></p>



 <div class="wp-block-ponyo-table">
    <table class="responsive-table">
        


 


                                                                                                    <tbody class="wp-block-ponyo-table-body">
    


 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Model Name</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Type</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Total Params</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Input Modalities</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Context Window</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Gemma 4 E2B</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Dense (Small)</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>5.1B</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Text, Vision, Audio</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>128K</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Gemma 4 E4B</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Dense (Small)</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>8.0B</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Text, Vision, Audio</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>128K</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Gemma 4 26B A4B</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>MoE</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>26.8B (3.8B active)</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Text, Vision</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>256K &#8211; 512K</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Gemma 4 31B</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Dense</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>31.3B</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Text, Vision</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>256K &#8211; 512K</span></p>


                    </span>
                                            </span>
            </td>


</tr> 

</tbody>


    </table>
</div>



<h2 class="wp-block-heading"><strong>Build the Future of AI with Docker Hub</strong></h2>



<p>The arrival of Gemma 4 on Docker Hub reinforces our commitment to making Docker Hub the best place to discover, share, and run AI models. Whether you are building a voice-activated mobile assistant or a large-scale document retrieval system, Docker Hub makes it simple to find the right model, pull it instantly, and run it anywhere.</p>



<p><strong>Ready?</strong> Head over to <a href="https://hub.docker.com/u/ai" id="dkr_docker-hub-88429" rel="nofollow noopener" target="_blank">Docker Hub</a> to pull the models<br /><br /><strong>Want to join the Docker Model Runner community?</strong> Please star, fork and contribute to our <a href="https://github.com/docker/model-runner" id="dkr_github-repo-88429" rel="nofollow noopener" target="_blank">GitHub repo</a></p>



<p></p>
