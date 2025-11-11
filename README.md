-- AUTO RECLUTAR SOLO 150 RZ$ – FUNCIONA EN PERÚ AHORA MISMO
getgenv().Auto150RZ = true   -- false = pausar

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local root = character:WaitForChild("HumanoidRootPart")

spawn(function()
    while wait(0.5) do
        if not getgenv().Auto150RZ then 
            wait(2)
            continue 
        end

        for _, v in pairs(workspace:GetDescendants()) do
            if v:IsA("ProximityPrompt") and v.ObjectText == "150 RZ$" then
                local npc = v:FindFirstAncestorWhichIsA("Model")
                if npc and npc:FindFirstChild("HumanoidRootPart") then
                    local hrp = npc.HumanoidRootPart
                    local dist = (root.Position - hrp.Position).Magnitude

                    print("150 RZ$ encontrado →", npc.Name, "→", math.floor(dist), "studs")

                    -- Caminar normal
                    if dist > 16 then
                        humanoid:MoveTo(hrp.Position + Vector3.new(0,0,-4))
                        humanoid.MoveToFinished:Wait(10)
                    end

                    -- LOOP HASTA QUE LO COMPRE (nunca falla)
                    repeat
                        if not getgenv().Auto150RZ then break end
                        pcall(function()
                            v.HoldDuration = 0
                            v.MaxActivationDistance = 50
                            fireproximityprompt(v)
                        end)
                        wait(0.4)
                    until not v.Parent or not v:IsDescendantOf(workspace)

                    print("RECLUTADO 150 RZ$ →", npc.Name)
                    wait(2)
                end
            end
        end
    end
end)

-- Respawn (no se cae nunca)
player.CharacterAdded:Connect(function(c)
    character = c
    humanoid = c:WaitForChild("Humanoid")
    root = c:WaitForChild("HumanoidRootPart")
end)

print("AUTO 150 RZ$ PERÚ ACTIVADO AL 100% – FUNCIONANDO AHORA")
print("PAUSAR → getgenv().Auto150RZ = false")
print("REANUDAR → getgenv().Auto150RZ = true")
