<script lang="ts">
	import {
		Button,
		DataTable,
		Dropdown,
		FileUploaderDropContainer,
		Form,
		FormGroup,
		FormLabel,
		ImageLoader,
		Modal,
		RadioButton,
		RadioButtonGroup,
		Search,
		Tab,
		TabContent,
		Tabs,
		TextArea,
		TextInput,
		ToastNotification,
		Toggle
	} from 'carbon-components-svelte';
	import './subscription.css';
	import type { each } from 'svelte/internal';

	let selectedRowIds:any[] = [];
	let files:File[] = [];
	let subscription_msg = '';
	let auto_renew_changed = true;
	let file_size_error = false;
	let users : any=[];
	$: subscription_history=[]

	async function fetchUrl(url:string) {
		const res = await fetch(url);
		const data = await res.json();

		if (res.ok) {
			return data;
		} else {
			throw new Error(data);
		}
	}

	async function getAllSubscriptionsData() {
		return fetchUrl('https://svapi.trusttalk.net/subscription_plans');
	}

	async function getUsers(){
		return fetchUrl('https://svapi.trusttalk.net/subscription/users');

	}



	function formateDate(dateString: string) {
		const dateObject = new Date(dateString);

		const options:object = { day: '2-digit', month: '2-digit', year: 'numeric' };
		const formattedDate = dateObject.toLocaleDateString('en-GB', options);
		return formattedDate;
	}

	let subscriptions_list = getAllSubscriptionsData();
	let users_list = getUsers();

	let i=0;

	$:{
		subscriptions_list
		.then(
			subscriptions=>{
				subscription_history = subscriptions
			}
			)
	}

	$:{
		users_list
		.then(users_=>{
			users=users_
		})
	}

	let open = false;

	let email_disabled = false;
	let search_disabled = false;

	let search_txt = '';
	let email_txt = '';

	let selected = 'trusttalkuser';

	$: {
		if (search_txt != '') {
			email_disabled = true;
		} else {
			email_disabled = false;
		}
	}

	$: {
		if (email_txt != '') {
			search_disabled = true;
		} else {
			search_disabled = false;
		}
	}


	let selectedFile: File;

	function handleFileChange(event : Event) {
		selectedFile = files[0];
	}

	$: {
		if (selectedFile) {
			const reader = new FileReader();

			reader.onload = () => {
				const preview : any = document.getElementById('preview') ;
				preview.src = reader.result;
			};

			reader.readAsDataURL(selectedFile);
		}
	}
</script>

<div class="main">
	<div class="main_content">
		<Tabs>
			<Tab label="Subscription Detail" />
			<Tab label="Customize" />
			<Tab label="Users" />
			<Tab label="History" />
			


			<svelte:fragment slot="content">
				<TabContent>
					{#await subscriptions_list}
						loading...
					{:then cs}
						<div class="tt_w_fit tt_m_center tt_mb_align_left tt_justify_center">
							<div class="tt_flex_w_100 flex tt_space_between tt_sp_flex">
								<div>
									<p>Subscription Start Date: <span>{formateDate(cs[0].start_date)}</span></p>
									<p>Subscription End Date: <span>{formateDate(cs[0].end_date)}</span></p>
									<p>Plan Users: <span>{cs[0].max_users_allowed}</span></p>
								</div>
								<div>
									<p class="flex align_center">
										Auto Renew: <span
											><Toggle
												on:change={() => {
													auto_renew_changed = !auto_renew_changed;
												}}
												toggled={cs[0].auto_recharge}
											/></span
										>
									</p>
									<p>Subscription Plan: <span>{cs[0].subscription_plan_name}</span></p>
									<p>Subscribed Users: <span>{cs[0].users_consumed}</span></p>
									<p>
										Available Users: <span>{cs[0].max_users_allowed - cs[0].users_consumed}</span>
									</p>
								</div>
							</div>
							<div class="tt_w_100">
								<Button disabled={auto_renew_changed} class="tt_btn" kind="primary">Save</Button>

							</div>
						</div>
					{/await}
				</TabContent>
				<TabContent>
					
					<div class="subscription_msg">
						<div class="tt_customize_flex">
							
							<TextArea
								class="tt_flex_1"
								placeholder="Subscription message for sponsers"
								rows={4}
								bind:value={subscription_msg}
								maxCount={30}
							/>

							<FileUploaderDropContainer
								on:change={handleFileChange}
								accept={['.jpg', '.jpeg', '.png']}
								class="file_upload"
								name="logo_file"
								bind:files
								multiple={false}
								validateFiles={(files) => {
									if (files[0].size > 1_024) {
										file_size_error = true;
										files = [];
										return files;
									}
									return files;
								}}
								labelText="Drag and drop your company logo or click to upload"
							/>
						</div>
						<div class="tt_preview">
							<FormLabel class="tt_header">Preview</FormLabel>
							<p>This is the preview of the message box that will appear to user.</p>
							<div class="tt_preview_box tt_mb_res">
								<ImageLoader class="tt_preview_img" src="/images/preview.png" />
								<div class="tt_msg_logo_box">
									{#if selectedFile}
										<img src="" alt="File Preview" width={30} id="preview" />
									{:else}
										<img src="images/user.png" width={30} alt="" />
									{/if}

									<p>{subscription_msg}</p>
								</div>
							</div>
						</div>
					</div>
					{#if file_size_error}
						<ToastNotification
							timeout={3000}
							class="tt_toast"
							lowContrast
							title="FileSizeError"
							subtitle="Please select file size less than 1 kb."
							caption={new Date().toLocaleString()}
							on:close={(e) => {
								e.preventDefault();
								file_size_error = false;
								// custom close logic (e.g., transitions)
							}}
						/>
					{/if}
				</TabContent>
				<TabContent>
					<div class="subscription_table">
						<DataTable
							selectable
							bind:selectedRowIds
							headers={[
								{ key: 'user_name', value: 'Name' },
								{ key: 'type', value: 'Type' },
								{ key: 'checked', value: 'Checked' }
							]}
							rows={users}
						>
							<svelte:fragment slot="cell" let:row let:cell>
								{#if cell.key === 'type'}
										Social
								{:else if cell.key === 'checked'}
									Checked
								{:else}
									{cell.value}
								{/if}
							</svelte:fragment>
						
						
						</DataTable>
						<Button on:click={() => (open = true)} class="tt_btn">Invite User</Button>
						<Button disabled={selectedRowIds.length == 0} class="tt_btn tt_btn_danger" kind="danger"
							>Revoke Subscription</Button
						>
					</div>
				</TabContent>
				<TabContent>
					<DataTable
							bind:selectedRowIds
							headers={[
								{ key: 'subscription_plan_name', value: 'Subscription Name' },
								{ key: 'start_date', value: 'Start Date' },
								{ key: 'end_date', value: 'End Date' }
							]}
							rows={subscription_history}
					>
					<svelte:fragment slot="cell" let:row let:cell>
						{#if cell.key === 'start_date' || cell.key === 'end_date'}
								{formateDate(cell.value)}
						{:else}
							{cell.value}
						{/if}
					</svelte:fragment>
				
					</DataTable>
				</TabContent>
			</svelte:fragment>
		</Tabs>
	</div>
</div>

<Modal class="model" passiveModal bind:open modalHeading="Search Trusttalk user" on:open on:close>
	<Form>
		<RadioButtonGroup
			class="radio_group"
			orientation="vertical"
			legendText="Search by"
			bind:selected
		>
			<RadioButton labelText="Trusttalk user name" value="trusttalkuser" />
			<RadioButton labelText="Email" value="email" />
			<RadioButton labelText="Linkedin Url" value="link" />
		</RadioButtonGroup>

		<!-- <Search disabled={search_disabled} bind:value={search_txt} placeholder="Search Trusttalk Users" /> -->

		<FormGroup class="email_invite">
			<TextInput
				bind:value={email_txt}
				disabled={email_disabled}
				class="email_input"
				type={selected == 'email' ? 'email' : 'text'}
				name="user"
				required
			/>
			<Button type="submit" class="invite_btn">Search User</Button>
		</FormGroup>
		<FormGroup class="email_invite">
			<div class="tt_user_info">
				<img src="images/user.png" alt="user" height="50" width="50" />
				<div>
					<h4>Ganesh Parmar</h4>
					<p>ganeshparmarms7@gmail.com</p>
				</div>
			</div>
			<Button type="submit" class="invite_btn">Send Invitation</Button>
		</FormGroup>
	</Form>
</Modal>

<style>
	.main {
		display: flex;
		justify-content: center;
		/* align-items: center; */
		height: 100vh;
	}
	.main_content {
		max-width: 900px;
		width: 100%;
		margin: 0 auto;
		padding: 1em;
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	.flex {
		display: flex;
		justify-content: space-evenly;
	}
	.tt_flex_w_100 {
		width: 100% !important;
	}

	.align_center {
		justify-content: space-between;
		align-items: center;
	}

	p {
		margin: 1em 0;
	}
	.subscription_table {
		margin-top: 2.5em;
	}

	@media (max-width: 768px) {
		.flex {
			display: block;
			max-width: 300px;
			width: 100%;
			margin: 0 auto;
		}
	}
</style>
