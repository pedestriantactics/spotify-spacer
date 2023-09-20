<script lang="ts">
  import SpotifyWebApi from "spotify-web-api-node";
  
  import eruda from 'eruda';
  eruda.init();

  const clientId = "dd146803015d46ed82fde33770c21e96";
  const clientSecret = "d5e6038902e6479a85d6bc8f947961b1";
  const redirectUri =
    "https://scaling-meme-5vj5qxrp764fvw75-5173.app.github.dev";

  const spotifyApi = new SpotifyWebApi({
    clientId: clientId,
    clientSecret: clientSecret,
    redirectUri: redirectUri,
  });

  const handleCallback = () => {
    const params = new URLSearchParams(window.location.search);
    const code = params.get("code");

    if (code) {
      spotifyApi
        .authorizationCodeGrant(code)
        .then((data) => {
          const accessToken = data.body.access_token;
          const refreshToken = data.body.refresh_token;

          spotifyApi.setAccessToken(accessToken);
          spotifyApi.setRefreshToken(refreshToken);

          spotifyApi
            .getUserPlaylists()
            .then((data) => {
              console.log("User playlists:", data.body.items);
              playlists = data.body.items.map((item) => ({
                id: item.id,
                name: item.name,
                tracks: item.tracks,
              }));
            })
            .catch((error) => {
              console.error("Error retrieving user playlists:", error);
            });
        })
        .catch((error) => {
          console.error("Error retrieving access token:", error);
        });
    }
  };

  handleCallback();

  const handleLogin = () => {
    const scopes = [
      "user-read-private",
      "user-read-email",
      "playlist-read-private",
    ];

    const authorizeUrl = `https://accounts.spotify.com/authorize?response_type=code&client_id=${clientId}&scope=${scopes.join(
      "%20"
    )}&redirect_uri=${redirectUri}`;

    window.location.replace(authorizeUrl);
  };

  // playlists

  let playlists = [] as Array<{ id: string; name: string; tracks: object }>;

  let selectedPlaylist = "";

  if (spotifyApi.getAccessToken()) {
    spotifyApi
      .getUserPlaylists()
      .then((data) => {
        console.log("User playlists:", data.body.items);
        playlists = data.body.items.map((item) => ({
          id: item.id,
          name: item.name,
          tracks: item.tracks,
        }));
      })
      .catch((error) => {
        console.error("Error retrieving user playlists:", error);
      });
  }

  // transport controls
  const handlePlay = () => {
    spotifyApi.play();
  };

  const handlePause = () => {
    spotifyApi.pause();
  };

  const handleNext = () => {
    spotifyApi.skipToNext();
  };

  const handlePrevious = () => {
    spotifyApi.skipToPrevious();
  };
</script>

<body>
  <!-- login -->
  <button on:click={handleLogin}>Log in with Spotify</button>

  <!-- playlist selector -->
  <select bind:value={selectedPlaylist}>
    {#each playlists as playlist}
      <option value={playlist.id}>{playlist.name}</option>
    {/each}
  </select>

  <!-- transport controls -->
  <button on:click={handlePlay}>Play</button>
  <button on:click={handlePause}>Pause</button>
  <button on:click={handleNext}>Next</button>
  <button on:click={handlePrevious}>Previous</button>
</body>
