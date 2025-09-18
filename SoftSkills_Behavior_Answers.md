# Câu Hỏi Phỏng Vấn: Soft Skills / Behavior

## 1. Kể một lần bạn gặp xung đột trong team. Bạn giải quyết thế nào?

**Tình huống:** 
Trong dự án trước, tôi và một developer khác có quan điểm khác nhau về cách implement authentication. Anh ấy muốn dùng JWT với thời hạn dài, còn tôi cho rằng nên implement refresh token để bảo mật tốt hơn.

**Cách giải quyết:**
- Tôi chủ động setup một cuộc meeting để thảo luận
- Chuẩn bị trước research về ưu/nhược điểm của cả 2 phương pháp
- Trình bày quan điểm của mình với data cụ thể về security risks
- Lắng nghe quan điểm của anh ấy về development speed và complexity
- Cùng nhau tìm compromise: implement JWT ngắn hạn + refresh token
- Document quyết định để team hiểu rõ lý do

**Kết quả:** 
Giải pháp cuối hoạt động tốt, team học được cách debate constructive và tôi build được relationship tốt hơn với colleague đó.

---

## 2. Khi deadline gấp mà bạn thấy giải pháp hiện tại chưa ổn, bạn xử lý thế nào?

**Approach của tôi:**
1. **Assess impact**: Đánh giá mức độ rủi ro nếu ship code hiện tại
2. **Communicate early**: Thông báo ngay với PM/Tech Lead về concerns
3. **Propose options**: 
   - Option A: Ship hiện tại + plan fix trong sprint sau
   - Option B: Delay để fix (nếu critical)
   - Option C: Implement quick workaround

**Ví dụ cụ thể:**
Lần trước có API response time chậm (2-3s) trước deadline. Tôi:
- Báo ngay với team về performance issue
- Implement caching layer nhanh trong 2 giờ
- Setup monitoring để track performance
- Document technical debt để refactor sau

**Principle:** Luôn prioritize communication và đưa ra solutions, không chỉ problems.

---

## 3. Bạn đã bao giờ refactor code lớn chưa? Lý do và kết quả?

**Project:** Refactor legacy authentication system từ session-based sang JWT

**Lý do:**
- Code cũ khó maintain (3000+ lines trong 1 file)
- Performance issues với session storage
- Khó scale horizontal
- Security vulnerabilities trong session handling

**Process:**
1. **Analysis**: Map toàn bộ auth flow hiện tại
2. **Planning**: Chia nhỏ thành 5 phases để không block development
3. **Testing**: Viết comprehensive tests trước khi refactor
4. **Implementation**: 
   - Phase 1: Tách logic thành modules nhỏ
   - Phase 2: Implement JWT service
   - Phase 3: Migrate routes từng bước
   - Phase 4: Update frontend
   - Phase 5: Remove old code

**Kết quả:**
- Response time cải thiện 40%
- Code coverage từ 30% lên 85%
- Bug reports giảm 60%
- Team velocity tăng do code dễ maintain hơn

---

## 4. Cách bạn học công nghệ mới?

**Process của tôi:**

1. **Understand the "Why"**
   - Tại sao công nghệ này ra đời?
   - Giải quyết vấn đề gì?
   - So sánh với alternatives

2. **Hands-on Learning**
   - Build một project nhỏ
   - Follow official documentation + tutorials
   - Join communities (Discord, Reddit, Stack Overflow)

3. **Deep Dive**
   - Đọc source code của thư viện
   - Understand internals và trade-offs
   - Tìm hiểu best practices

4. **Teaching & Sharing**
   - Viết blog posts hoặc tạo internal docs
   - Share với team hoặc mentor juniors
   - Present trong team meetings

**Ví dụ:** Khi học GraphQL
- Bắt đầu với simple queries
- Build blog app với Apollo Client/Server
- Research về N+1 problem và caching strategies
- Viết comparison document GraphQL vs REST cho team

---

## 5. Bạn muốn phát triển sự nghiệp thế nào trong 2-3 năm tới?

**Roadmap ngắn hạn (1 năm):**
- **Technical**: Master system design patterns, cloud architecture (AWS/GCP)
- **Leadership**: Lead 2-3 junior developers, mentor newcomers
- **Domain**: Chuyên sâu về performance optimization và security

**Roadmap trung hạn (2-3 năm):**
- **Role evolution**: Transition từ Senior Developer sang Tech Lead
- **Skills expansion**: 
  - DevOps practices (CI/CD, Infrastructure as Code)
  - Product thinking và business understanding
  - Cross-team collaboration
- **Impact**: Lead technical decisions cho team 8-10 người

**Cách achieve:**
- Tham gia các technical committees
- Contribute vào architecture decisions
- Build internal tools để improve developer experience  
- Take ownership của critical systems
- Continuous learning thông qua conferences, courses

**Tại sao chọn company này:**
- Culture of learning và innovation
- Opportunity để work với scale problems
- Strong engineering practices
- Room for growth và career advancement

---

## Tips Khi Trả Lời Câu Hỏi Behavior

1. **STAR Method**: Situation, Task, Action, Result
2. **Be specific**: Đưa ra examples cụ thể, không nói chung chung
3. **Show growth**: Demonstrate learning từ mistakes
4. **Quantify results**: Dùng numbers khi có thể
5. **Be honest**: Thừa nhận limitations nhưng show cách overcome
6. **Ask clarifying questions**: Nếu câu hỏi unclear, ask for context

## Red Flags Cần Tránh

❌ Blame teammates hoặc managers  
❌ Nói không có weaknesses  
❌ Answers quá generic  
❌ Không prepare examples trước  
❌ Focus quá nhiều vào technical mà quên soft skills  
❌ Không show passion for learning
