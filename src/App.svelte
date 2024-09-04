<script lang="ts">
  import "./app.css";
  import { ModeWatcher } from "mode-watcher";
  import { Button } from "$lib/components/ui/button";
  import { Input } from "$lib/components/ui/input";
  import { Card, CardContent, CardHeader, CardTitle } from "$lib/components/ui/card";
  import { Skeleton } from "$lib/components/ui/skeleton";
  import {Bolt, LoaderCircle} from "lucide-svelte";
  import * as Collapsible from "$lib/components/ui/collapsible";
  import * as Select from "$lib/components/ui/select";
  import * as Avatar from "$lib/components/ui/avatar";
  import Footer from "$lib/widgets/Footer.svelte";
  import Theme from "$lib/widgets/Theme.svelte";

  const language = [
    { value: "auto", label: "Auto" },
    { value: "en", label: "English" },
    { value: "id", label: "Indonesia" }
  ]

  type resultType = {
    username: string;
    generated_content: string;
    avatar_url: string;
  }

  let loading : boolean = false;
  let username : string = "";
  let lang : string = "auto";
  let result : resultType = {
    username: "",
    generated_content: "",
    avatar_url: ""
  }
  let key : string = "";
  let errorMessage = "";
  const apiUrl = "https://roast.savioruz.me/api/v1/roast";

  function validateInput() {
    if (!username.trim()) {
      errorMessage = "Username is required.";
      return false;
    } else if (username.length < 6 || username.length > 32) {
      errorMessage = "Username must be between 6 and 32 characters.";
      return false;
    }
    errorMessage = "";
    return true;
  }

  async function handleRoast() {
    loading = true;
    if (!validateInput()) {
      loading = false;
      return;
    }

    if (document.getElementById("lang")) {
      lang = (document.getElementById("lang") as HTMLInputElement).value;
    } else {
      lang = "auto";
    }

    if (document.getElementById("key")) {
      key = (document.getElementById("key") as HTMLInputElement).value;
    }

    const myHeaders = new Headers();
    myHeaders.append("Content-Type", "application/json");

    const raw = JSON.stringify({
      "username": username,
      "lang": lang,
      "key": key
    });

    const requestOptions = {
      method: "POST",
      headers: myHeaders,
      body: raw
    };

    try {
      const response = await fetch(apiUrl, requestOptions);
      if (response.ok) {
        const data = await response.json();
        result.generated_content = data.data.generated_content.replace(/\n/g, "<br>").replace(/\*/g, "");
        result.username = data.data.username;
        result.avatar_url = data.data.avatar_url;
      } else if (response.status === 400) {
        throw new Error("Invalid request.");
      } else {
        throw new Error("Profile not found or API error. Please try again later.");
      }
    } catch (error) {
      console.error(error);
      errorMessage = error as string;
    } finally {
      loading = false;
    }
  }
</script>

<ModeWatcher />
<main class="flex-grow container mx-auto px-4 py-16 max-w-4xl min-h-[90vh]">
  <Card class="mb-12 shadow-lg">
    <CardHeader>
      <CardTitle>Roastgithub</CardTitle>
    </CardHeader>
    <CardContent>
      <div class="flex flex-col space-y-4">
        <Input id="username" bind:value={username} type="text" placeholder="Enter your username" required />
        {#if loading}
          <Button disabled>
            <LoaderCircle class="mr-2 h-4 w-4 animate-spin" />
            Please wait
          </Button>
        {:else}
          <Button on:click={handleRoast}>Roast</Button>
        {/if}
      </div>
      {#if loading}
        <div class="p-4 my-3 space-y-5">
          <Skeleton class="h-[35px] max-w-[150px] md:max-w-sm" />
          <div class="space-y-3">
            <Skeleton class="h-[25px] max-w-[200px] md:max-w-lg" />
            <Skeleton class="h-[25px]" />
            <Skeleton class="h-[25px]" />
          </div>
        </div>
      {:else if result.generated_content}
        <div class="p-4 my-3">
          <div class="flex justify-between">
            <h2 class="text-lg text-gray-500">Roasting @{result.username}</h2>
            <Avatar.Root>
              <Avatar.Image src={result.avatar_url} alt="{result.username}" />
              <Avatar.Fallback>avatar</Avatar.Fallback>
            </Avatar.Root>
          </div>
          <br>
          <p>{@html result.generated_content}</p>
        </div>
      {/if}
      {#if errorMessage}
        <div class="bg-red-100 text-red-800 rounded-sm mb-4 mt-4 p-3 w-fit">
          <p class="text-sm">{errorMessage}</p>
        </div>
      {/if}
    </CardContent>
  </Card>
    <div class="flex">
      <Collapsible.Root class="space-y-3 basis-full">
        <div class="flex items-center space-x-4 px-4">
          <Collapsible.Trigger asChild let:builder>
            <Button builders={[builder]} variant="ghost">
              <Bolt class="mr-2 h-4 w-4" />
              Settings
            </Button>
          </Collapsible.Trigger>
        </div>
        <Collapsible.Content class="space-y-3 px-4">
          <Select.Root portal={null}>
            <Select.Trigger class="w-[180px]">
              <Select.Value placeholder="Select a language" />
            </Select.Trigger>
            <Select.Content>
              <Select.Group>
                <Select.Label>Language</Select.Label>
                {#each language as l}
                  <Select.Item value={l.value} label={l.label}
                  >{l.label}</Select.Item
                  >
                {/each}
              </Select.Group>
            </Select.Content>
            <Select.Input id="lang" name="lang" />
          </Select.Root>
          <Input id="key" bind:value={key} type="text" placeholder="Key" class="w-full" />
        </Collapsible.Content>
      </Collapsible.Root>
      <div class="basis-0 mr-4">
        <Theme />
      </div>
    </div>
</main>
<Footer />
