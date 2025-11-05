---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

Xin chào, mình là Cao Minh Quang 👋    
  
 Sinh viên năm cuối tại **Trường Đại học Khoa học Tự nhiên – Đại học Quốc gia TP. Hồ Chí Minh (HCMUS)** 🎓


<div style="text-align:center;">
<p id="typing-text" style="font-family:'Courier New', monospace; font-size:1.1rem; color:#2b6cb0; display:inline-block; white-space:pre;"></p>
</div>

<script>
  const text = "University of Science - VNUHCM - Hồ Chí Minh";
  const el = document.getElementById("typing-text");
  let i = 0;

  function type() {
    if (i < text.length) {
      el.textContent += text.charAt(i);
      i++;
      setTimeout(type, 80);
    } else {
      el.insertAdjacentHTML("beforeend", '<span style="animation: blink 1s infinite;">|</span>');
    }
  }

  document.addEventListener("DOMContentLoaded", type);

  const style = document.createElement("style");
  style.textContent = `
    @keyframes blink { 
      0%, 50%, 100% { opacity: 1; } 
      25%, 75% { opacity: 0; } 
    }
  `;
  document.head.appendChild(style);
</script>

<!-- <div style="text-align:center; margin-top:60px;">
  <p style="font-style:italic; font-size:1rem; color:#555;">
    
  </p>
</div> -->

Blog này được xây dựng như một **nhật ký học tập và sáng tạo cá nhân**, nơi mình ghi lại:
- Những bài học, lỗi sai và kinh nghiệm thực tế khi lập trình.  
- Các ý tưởng nghiên cứu và công nghệ mà tôi đang tìm hiểu.  
- Các dự án nhỏ nhưng chứa đựng niềm đam mê với lập trình và tri thức.


Bản thân mình cũng chỉ là một người đam mê công nghệ đang học hỏi và tìm hiểu. Nếu bạn cũng đang theo đuổi hành trình công nghệ, sáng tạo và có các kiến thức muốn trao đổi, hãy kết nối với mình qua các kênh thông tin: 

- **Email:** [mquangcao38@gmail.com](mailto:mquangcao38@gmail.com)
- **LinkedIn:** [linkedin.com/in/mquangcao](https://www.linkedin.com/in/mquangcao/)
