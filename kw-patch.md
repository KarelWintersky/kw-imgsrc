# dialog.php

Старый код:

    if (isset($_GET['editor']))
    {
        $editor = strip_tags($_GET['editor']);
    } else {
        if($_GET['type']==0){
            $editor=false;
        } else {
            $editor='tinymce';
        }
    }

Новый код:

    if (isset($_GET['editor']))
    {
        $editor = strip_tags($_GET['editor']);
    } else {
        $editor = false;
    }

# include.js

Разворачиваем минифицированную версию и находим кусочек:

    var editor = $('#editor').val();
    if (editor == 'ckeditor')
    {
        var funcNum = getUrlParam('CKEditorFuncNum');
        window.opener.CKEDITOR.tools.callFunction(funcNum, url);
        window.close();
    } else {
        if (parent.tinymce.majorVersion < 4) {
            parent.tinymce.activeEditor.windowManager.params.setUrl(url);
            parent.tinymce.activeEditor.windowManager.close(parent.tinymce.activeEditor.windowManager.params.mce_window_id);
        } else {
            // tinymce 4.X
            parent.tinymce.activeEditor.windowManager.getParams().setUrl(url);
            parent.tinymce.activeEditor.windowManager.close();
        }
    }

Этот кусочек кода отрабатывает только если мы в вызове dialog.php установили type отличный от 0.
Однако автор наивно предполагает, что если у нас не CKEditor - то уж точно TinyMCE.

Это не так :)

Поэтому новый код таков:

			var editor = $('#editor').val();
			if (editor == 'ckeditor')
			{
				var funcNum = getUrlParam('CKEditorFuncNum');
				window.opener.CKEDITOR.tools.callFunction(funcNum, url);
				window.close();
			}
			else
			{
                // KW Patch for lightbox image on LMB and copy url to clipboard
                // lightbox this image
                $('#full-img').attr('src', decodeURIComponent(url));
                $('#previewLightbox').lightbox();

                // copy url to clipboard
                url = url.substring(0, url.indexOf('?'));
                var targetId = "_hiddenCopyText_";
                target = document.getElementById(targetId);
                if (!target) {
                    var target = document.createElement("textarea");
                    target.style.position = "absolute";
                    target.style.left = "-9999px";
                    target.style.top = "0";
                    target.id = targetId;
                    document.body.appendChild(target);
                }
                target.textContent = url;
                var currentFocus = document.activeElement;
                target.focus();
                target.setSelectionRange(0, target.value.length);
                var succeed;
                try {
                    succeed = document.execCommand("copy");
                } catch(e) {
                    succeed = false;
                }
                // restore original focus
                if (currentFocus && typeof currentFocus.focus === "function") {
                    currentFocus.focus();
                }
                target.textContent = "";
                // KW Patch end
			}

