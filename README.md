<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>내 블로그</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
            color: #333;
            transition: background 0.3s, color 0.3s;
        }
        header {
            background: #333;
            color: white;
            padding: 20px;
            text-align: center;
        }
        .container {
            width: 80%;
            margin: 20px auto;
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
        }
        .post-list {
            list-style: none;
            padding: 0;
        }
        .post-list li {
            background: #ddd;
            margin: 10px 0;
            padding: 10px;
            border-radius: 5px;
            cursor: pointer;
        }
        .post-list li:hover {
            background: #bbb;
        }
        .comment-section {
            margin-top: 20px;
        }
        .comment-section input, .comment-section button {
            padding: 10px;
            font-size: 16px;
        }
        .comment-section input {
            width: 80%;
        }
        .comment-section button {
            background-color: #333;
            color: white;
            border: none;
            cursor: pointer;
        }
        .comment-list {
            margin-top: 10px;
            padding: 10px;
            background: #eee;
            border-radius: 5px;
        }
        .hidden {
            display: none;
        }
        .like-button {
            background: none;
            border: none;
            cursor: pointer;
            font-size: 18px;
        }
        .dark-mode {
            background-color: #222;
            color: #fff;
        }
        .dark-mode .container {
            background: #333;
            color: #fff;
        }
        .dark-mode .post-list li {
            background: #555;
            color: white;
        }
        .dark-mode .post-list li:hover {
            background: #777;
        }
        .dark-mode .comment-list {
            background: #444;
        }
        img {
            max-width: 100%;
            margin-top: 20px;
        }
        .pagination {
            text-align: center;
            margin-top: 20px;
        }
        .pagination button {
            padding: 10px;
            background-color: #333;
            color: white;
            border: none;
            cursor: pointer;
            margin: 0 5px;
        }
        .scheduler {
            margin-top: 20px;
        }
    </style>
</head>
<body>

<header>
    <h1>내 블로그</h1>
    <button onclick="toggleDarkMode()">🌙 다크 모드</button>
</header>

<div class="container">
    <h2>📌 게시글 목록</h2>
    <ul class="post-list" id="postList">
        <!-- 게시글 목록 -->
    </ul>

    <!-- 페이징 처리 -->
    <div class="pagination" id="pagination">
        <!-- 페이지 번호 버튼들 -->
    </div>

    <!-- 게시글 작성 폼 -->
    <h2>✍️ 새 게시글 작성</h2>
    <input type="text" id="newPostTitle" placeholder="제목 입력">
    <br><br>
    <textarea id="newPostContent" rows="4" cols="50" placeholder="내용 입력"></textarea>
    <br><br>
    <input type="file" id="newPostImage" accept="image/*">
    <br><br>
    
    <!-- 게시글 작성 시 게시 날짜 설정 -->
    <div class="scheduler">
        <label for="postDate">게시 날짜 및 시간:</label>
        <input type="datetime-local" id="postDate">
    </div>
    <br><br>
    <button onclick="addNewPost()">게시글 추가</button>

    <!-- 게시글 내용 표시 -->
    <div id="postContent" class="hidden">
        <h2 id="postTitle"></h2>
        <p id="postBody"></p>
        <img id="postImage" class="hidden">
        <button class="like-button" onclick="toggleLike()">❤️ <span id="likeCount">0</span></button>

        <div class="comment-section">
            <h3>댓글 남기기</h3>
            <input type="text" id="commentInput" placeholder="댓글을 입력하세요">
            <button onclick="addComment()">등록</button>
            <div id="commentList" class="comment-list"></div>
        </div>
    </div>
</div>

<footer>
    &copy; 2025 내 블로그 - 모든 권리 보유
</footer>

<script>
    let posts = [];
    let currentPostId = null;
    let likeCounts = {};
    let postsPerPage = 5;  // 한 페이지에 보여줄 게시글 수
    let currentPage = 1;   // 현재 페이지 번호

    // 게시글 목록 렌더링
    function renderPostList() {
        let postList = document.getElementById("postList");
        postList.innerHTML = "";

        let startIndex = (currentPage - 1) * postsPerPage;
        let endIndex = startIndex + postsPerPage;
        let paginatedPosts = posts.slice(startIndex, endIndex);

        paginatedPosts.forEach(post => {
            let li = document.createElement("li");
            li.textContent = post.title;
            li.onclick = () => showPost(post.id);
            postList.appendChild(li);
        });

        renderPagination();
    }

    // 게시글 페이지 번호 렌더링
    function renderPagination() {
        let pagination = document.getElementById("pagination");
        pagination.innerHTML = "";

        let totalPages = Math.ceil(posts.length / postsPerPage);
        
        if (currentPage > 1) {
            let prevButton = document.createElement("button");
            prevButton.textContent = "이전";
            prevButton.onclick = () => changePage(currentPage - 1);
            pagination.appendChild(prevButton);
        }

        for (let i = 1; i <= totalPages; i++) {
            let pageButton = document.createElement("button");
            pageButton.textContent = i;
            if (i === currentPage) {
                pageButton.disabled = true;
            }
            pageButton.onclick = () => changePage(i);
            pagination.appendChild(pageButton);
        }

        if (currentPage < totalPages) {
            let nextButton = document.createElement("button");
            nextButton.textContent = "다음";
            nextButton.onclick = () => changePage(currentPage + 1);
            pagination.appendChild(nextButton);
        }
    }

    // 페이지 변경
    function changePage(page) {
        currentPage = page;
        renderPostList();
    }

    // 게시글 표시
    function showPost(postId) {
        let post = posts.find(p => p.id === postId);
        if (!post) return;

        currentPostId = postId;
        document.getElementById("postTitle").textContent = post.title;
        document.getElementById("postBody").textContent = post.content;
        document.getElementById("postImage").classList.add("hidden");
        if (post.image) {
            let img = document.getElementById("postImage");
            img.src = post.image;
            img.classList.remove("hidden");
        }
        document.getElementById("postContent").classList.remove("hidden");

        let likeCount = likeCounts[postId] || 0;
        document.getElementById("likeCount").textContent = likeCount;
        document.getElementById("commentList").innerHTML = "";
    }

    // 게시글 추가
    function addNewPost() {
        let title = document.getElementById("newPostTitle").value.trim();
        let content = document.getElementById("newPostContent").value.trim();
        let imageFile = document.getElementById("newPostImage").files[0];
        let postDate = document.getElementById("postDate").value;

        if (title === "" || content === "") {
            alert("제목과 내용을 입력하세요!");
            return;
        }

        let postDateObj = new Date(postDate);
        let now = new Date();

        // 게시 날짜가 현재 날짜 이전이면 게시 불가
        if (postDateObj <= now) {
            alert("미래 날짜를 입력하세요!");
            return;
        }

        let image = null;
        if (imageFile) {
            let reader = new FileReader();
            reader.onload = function(e) {
                image = e.target.result;
                let newPost = { id: posts.length + 1, title, content, image, postDate: postDateObj };
                posts.push(newPost);
                sortAndRenderPosts();
            };
            reader.readAsDataURL(imageFile);
        } else {
            let newPost = { id: posts.length + 1, title, content, image, postDate: postDateObj };
            posts.push(newPost);
            sortAndRenderPosts();
        }

        document.getElementById("newPostTitle").value = "";
        document.getElementById("newPostContent").value = "";
        document.getElementById("newPostImage").value = "";
        document.getElementById("postDate").value = "";
    }

    // 게시글 목록 정렬 및 렌더링
    function sortAndRenderPosts() {
        posts.sort((a, b) => a.postDate - b.postDate); // 게시일 기준 정렬
        renderPostList();
    }

    // 댓글 추가
    function addComment() {
        let commentInput = document.getElementById("commentInput");
        if (commentInput.value.trim() === "") {
            alert("댓글을 입력하세요!");
            return;
        }
        let commentList = document.getElementById("commentList");
        let newComment = document.createElement("p");
        newComment.textContent = commentInput.value;
        commentList.appendChild(newComment);
        commentInput.value = "";
    }

    // 좋아요 토글
    function toggleLike() {
        if (currentPostId === null) return;
        if (!likeCounts[currentPostId]) {
            likeCounts[currentPostId] = 0;
        }
        likeCounts[currentPostId]++;
        renderPostList();
        showPost(currentPostId);
    }

    // 다크모드 토글
    function toggleDarkMode() {
        document.body.classList.toggle("dark-mode");
    }

    // 처음에 페이지 로드 시 게시글 렌더링
    window.onload = function() {
        renderPostList();
    }
</script>

</body>
</html>
