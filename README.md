# Restored support after [ox_inventory](https://github.com/CommunityOx/ox_inventory) support was restored

# 🔄 ESX to ox_core Conversion Guide

This guide helps you convert your legacy **ESX-based** FiveM scripts to the modern, performant **ox_core** framework.

---

## 📦 Requirements

- [ox_core](https://github.com/overextended/ox_core)
- [ox_lib](https://github.com/overextended/ox_lib)
- [ox_inventory](https://github.com/overextended/ox_inventory)
- [oxmysql](https://github.com/overextended/oxmysql) for database handling

---

## 🧠 Core Differences

| ESX                             | ox_core / ox_lib                                     |
|----------------------------------|------------------------------------------------------|
| `ESX = nil` & shared object      | ❌ Not needed — use modular exports and callbacks     |
| `ESX.RegisterUsableItem`        | ✅ `ox_inventory:useItem` event                      |
| `xPlayer.getMoney()`            | ✅ `xPlayer.get('money')`                            |
| `ESX.RegisterServerCallback`    | ✅ `lib.callback.register`                           |
| `ESX.TriggerServerCallback`     | ✅ `lib.callback.await()`                            |
| `ESX.UI.Menu.Open(...)`         | ✅ `lib.registerContext()` (for radial/inputs/etc.)  |
| `esx:getSharedObject` event     | ❌ Replaced by modular design                        |

---

## 🧩 Exports Reference

### ✅ ox_core (Server)
```lua
local xPlayer = exports.ox_core:getPlayer(source)

xPlayer.get('money')  
xPlayer.add('money', 500)
xPlayer.remove('item', 'bread', 1)
xPlayer.hasItem('lockpick', 1)

✅ ox_core (Client)

local playerData = LocalPlayer.state
print(playerData.groups) -- Check job/permissions

🧰 Callbacks (Server + Client)
✅ Server → Client Callback

-- Server
lib.callback.register('myresource:getName', function(source, args)
    return 'JaanMangib'
end)

-- Client
local result = lib.callback.await('myresource:getName', false)
print(result) -- "JaanMangib"

🍞 Usable Items
ESX Version

ESX.RegisterUsableItem('bread', function(source)
    local xPlayer = ESX.GetPlayerFromId(source)
    xPlayer.removeInventoryItem('bread', 1)
end)

ox_inventory Version

RegisterNetEvent('ox_inventory:useItem', function(item)
    if item.name ~= 'bread' then return end

    local xPlayer = exports.ox_core:getPlayer(source)
    xPlayer.removeInventoryItem('bread', 1)
    TriggerClientEvent('myresource:eat', source)
end)

🎯 Target Integration (ox_target)

exports.ox_target:addBoxZone({
    coords = vec3(123.4, -321.0, 28.7),
    size = vec3(2, 2, 2),
    rotation = 45,
    debug = false,
    options = {
        {
            name = 'npc_shop',
            icon = 'fas fa-store',
            label = 'Open Shop',
            onSelect = function(data)
                TriggerEvent('myresource:openShop')
            end
        }
    }
})

📋 Menus (ox_lib Context Menu)

lib.registerContext({
    id = 'job_menu',
    title = 'Job Menu',
    options = {
        {
            title = 'Get Uniform',
            icon = 'shirt',
            onSelect = function()
                TriggerServerEvent('myresource:getUniform')
            end
        },
        {
            title = 'Clock In',
            onSelect = function()
                TriggerServerEvent('myresource:clockIn')
            end
        }
    }
})

-- To open the menu:
lib.showContext('job_menu')

🧾 Input Dialogs (ox_lib)

local input = lib.inputDialog('Enter Info', {
    { type = 'input', label = 'Name', required = true },
    { type = 'number', label = 'Amount', required = true },
})

if input then
    print(input[1], input[2])
end

📡 Notifications

lib.notify({
    title = 'Success',
    description = 'Item purchased!',
    type = 'success'
})

🗃️ SQL Migration (MySQL to oxmysql)
ESX

MySQL.Async.fetchAll('SELECT * FROM users', {}, function(result)
end)

oxmysql

local result = MySQL.query.await('SELECT * FROM users WHERE identifier = ?', { identifier })

✅ Recommended Folder Structure

my-resource/
├── fxmanifest.lua
├── client/
│   └── main.lua
├── server/
│   └── main.lua
├── shared/
│   └── config.lua
└── README.md

⚙️ fxmanifest.lua Template

fx_version 'cerulean'
game 'gta5'

description 'ESX to ox_core converted script'
author 'YourName'

shared_scripts {
    '@ox_lib/init.lua',
    'shared/config.lua'
}

client_scripts {
    'client/main.lua'
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/main.lua'
}

lua54 'yes'

🧪 Final Notes

    Always test each component after converting

    Replace UI menus, notifications, and inputs with ox_lib

    Avoid any ESX-specific events or triggers — ox_core is modular and cleaner

👷 Need Help?

Check the official Overextended Docs:
📚 https://overextended.dev/

Or open an issue in this repository!
