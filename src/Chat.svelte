<script>
    import Login from './Login.svelte';
    import ChatMessage from './ChatMessage.svelte';
    import {username,user} from './User';
    import {onMount} from 'svelte';

    import GUN from 'gun';
    const db = GUN();

    let newMessage;
    let messages = [];

    onMount(() => {

        // Get Messages
        db.get('chat')
            .map()
            .once(async (data,id) => {
                if (data) {
                    //Key for E2E encryption
                    const key = "#foo";

                    var message = {
                        //Transform the data
                        who: await db.user(data).get('alias'), // a user might lie who they are! so let the user system determine
                        what: (await SEA.decrypt(data.what,key))+'', // force decrypt as text
                        when: GUN.state.is(data,'what'), // get the internal timestamp for the what property
                    };

                    if (message.what) {
                        messages = [...messages.slice(-100),message]
                    }
                }
            });
    });

    async function sendMessage() {
        const secret = await SEA.encrypt(newMessage, '#foo');
        const message = user.get('all').set({what: secret});
        const index = new Date().toISOString();
        db.get('chat').get(index).put(message);
        newMessage = '';
    }

</script>

<div class="container">
    {#if $username}
    <main>
        {#each messages as message (message.when)}
            <ChatMessage {message} sender={$username} />
        {/each}

    </main>
    <form on:submit|preventDefault={sendMessage}></form>
    {:else}
    <main>
        <Login />
    </main>
    {/if}
</div>