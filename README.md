-- [[ SISTEMA DE PROTEÇÃO E EXECUÇÃO ]] --

local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterGui = game:GetService("StarterGui")
local LocalPlayer = Players.LocalPlayer



    -- ==========================================
    -- EXECUÇÃO DO SCRIPT PRINCIPAL (DUPE)
    -- ==========================================
    pcall(function()
        local Backpack = LocalPlayer:WaitForChild("Backpack")

        -- Bypass Notify
        pcall(function()
            if ReplicatedStorage:FindFirstChild("NotifyClient") then
                for _, conn in pairs(getconnections(ReplicatedStorage.NotifyClient.Event)) do
                    conn:Disable()
                end
            end
            if ReplicatedStorage:FindFirstChild("Notify") then
                for _, conn in pairs(getconnections(ReplicatedStorage.Notify.OnClientEvent)) do
                    conn:Disable()
                end
            end
        end)

        local function notify(text)
            StarterGui:SetCore("SendNotification", {
                Title = "Sistema",
                Text = text,
                Duration = 4
            })
        end

        notify("Whitelist Aceita! Equipe o item.")

        local InvRemote = ReplicatedStorage.Modules.InvRemotes.InvRequest

        -- Hook do Remote para Duplicação
        local oldNamecall
        oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
            local args = {...}
            local method = getnamecallmethod()

            if self == InvRemote and method == "InvokeServer" then
                local result = oldNamecall(self, ...)
                task.wait(0.15)
                pcall(function()
                    self:InvokeServer(unpack(args))
                end)
                notify("Item duplicado! Guarde ou passe para alguém.")
                return result
            end
            return oldNamecall(self, ...)
        end)

        -- Monitor de Inventário Nil (Re-equipar)
        local lastTools = #Backpack:GetChildren()
        task.spawn(function()
            while true do
                task.wait(0.4)
                local currentTools = #Backpack:GetChildren()
                if currentTools < lastTools then
                    local tool = Backpack:FindFirstChildOfClass("Tool")
                    if tool then
                        LocalPlayer.Character.Humanoid:EquipTool(tool)
                    end
                end
                lastTools = currentTools
            end
        end)
    end)

else
    LocalPlayer:Kick("Rede não autorizada para este script.")
end
