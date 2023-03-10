<script lang="ts">
	import { enhance, applyAction } from '$app/forms';
	import { marked } from 'marked';
	import { pb, currentUser } from '$lib/pocketbase';
	import type { postErrorType } from '../../../types/types';
	import DOMPurify from 'isomorphic-dompurify';
	import toast from 'svelte-french-toast';
	import { slugify } from '$lib/utils';

	marked.setOptions({
		renderer: new marked.Renderer(),
		gfm: true,
		breaks: true,
		sanitize: false,
		xhtml: false,
		smartypants: true
	});

	const renderer = {
		image(href: string, title: string, text: string) {
			return `
				<img src="${href}" alt="${text}" style="width: 100%; height: 300px; object-fit: cover" />
			`;
		}
	};

	marked.use({ renderer });

	

	const author = $currentUser?.name.length > 0 ? $currentUser?.name : $currentUser?.username;

	function test(e: Event) {
		const target = e.currentTarget as HTMLInputElement;
		let file: File | null = target.files![0];

		const bannerImg: HTMLImageElement | null = document.querySelector('#banner-uploaded');
		let reader = new FileReader();
		reader.onload = function (e) {
			bannerImg!.src = e.target?.result as string;
		};
		reader.readAsDataURL(file);
	}

	export let form: postErrorType;
    export let data: any;

    const post = data.post;

    let title = post.title;
	let content = post.content;
</script>

<svelte:head>
	<link
		href="https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;500;600;700&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div class="w-screen h-full">
	<form
		action=""
		method="POST"
		class="h-[92vh]"
		enctype="multipart/form-data"
		use:enhance={() => {
			return async ({ result, update }) => {
				pb.authStore.loadFromCookie(document.cookie);
				await applyAction(result);

				switch (result.type) {
					case 'success':
						await update();
						break;
					case 'failure':
						Object.values(form.errors).forEach((error) => {
							toast.error(error[0]);
						});
						await update();
						break;
					case 'error':
						toast.error(result.error.message);
						break;
					default:
						await update();
				}
			};
		}}
	>
		<div class="w-full h-full flex flex-col items-center">
			
			<div class="flex w-full h-fit prose max-w-full items-center">
				<input type="text" name="title" id="title" placeholder="Enter your title here" class="text-4xl font-black text-black placeholder:text-black/70 bg-[#9fa8b6] focus-visible:outline-none w-1/2 p-[5vh]" bind:value={title}  autocomplete="off"/>
				<h1 class="w-1/2 p-[5vh] h-full border-l-[0.3vh] border-black">{title}</h1>
			</div>
			<div class="h-full w-full max-w-full flex items-center overflow-auto">
				<textarea name="content" id="content" placeholder="Enter your content here (Markdown supported)" class="placeholder:text-black/60 resize-none text-[1.6vh] bg-[#9fa8b6] focus-visible:outline-none h-full w-1/2 px-[5vh]" bind:value={content} />
				<div class="overflow-auto prose prose-neutral lg:prose-xl prose-p:after:content-[''] prose-p:before:content-[''] prose-blockquote:border-l-black h-full w-1/2 max-w-full px-[5vh] pb-[5vh] border-l-[0.3vh] border-black">
					<img src={`http://127.0.0.1:8090/api/files/${post.collectionId}/${post.id}/${post.picture}`} alt="" id="banner-uploaded" />
					{@html DOMPurify.sanitize(
						marked.parse(content.replace(/^[\u200B\u200C\u200D\u200E\u200F\uFEFF]/, ''))
					)}
				</div>	
			</div>
		</div>

		<div class="h-[8vh] w-full border-t-2 border-black flex justify-between items-center px-[1.5vh]">
			<div>
				<label for="picture" class="custom-picture"> Modifiy the banner of your post </label>
				<input id="picture" name="picture" accept="image/*" type="file" on:change={test} />
			</div>
			<button type="submit">Post</button>
		</div>

		<input type="text" name="author" hidden value={author} />
		<input type="text" name="slug" hidden value={slugify(title)} />
	</form>
</div>

<style>

	:global(body) {
		background-color: #9fa8b6;
	}

	button {
		padding: 1.2vh 3vh;
		background-color: #9fa8b6;
		border: 2px solid black;
		transition: background-color 0.15s ease;
		font-size: 2vh;
	}

	button:hover {
		background-color: black;
		color: #9fa8b6;
		cursor: pointer;
	}

	button:active {
		background-color: #9fa8b6;
		color: #000;
		cursor: pointer;
	}

	input[type='file'] {
		display: none;
	}
	.custom-picture {
		border: 2px solid #000;
		display: inline-block;
		padding: 1.2vh 3vh;
		cursor: pointer;
		font-size: 2vh;
		transition: background-color 0.15s ease, color 0.15s ease;
	}

	.custom-picture:hover {
		background-color: black;
		color: #9fa8b6;
		cursor: pointer;
	}

	.custom-picture:active {
		background-color: #9fa8b6;
		color: #000;
		cursor: pointer;
	}

	#banner-uploaded {
		width: 100%;
		height: 30vh;
		object-fit: cover;
		margin-bottom: 2vh;
	}
</style>
