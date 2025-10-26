dll 코드(ModBehaviour.cs) 전문입니다.
참고하시기 바랍니다.


        using UnityEngine;
        using Duckov.Modding;          
        using SodaCraft.Localizations; 
        using System;
        using System.IO;                 
        using System.Text;               
        using System.Linq;             

    namespace KoreanPatch
        {
    public class ModBehaviour : Duckov.Modding.ModBehaviour
    {
        private string modDirectoryPath;

        public void Awake()
        {
            try
            {
                modDirectoryPath = Path.GetDirectoryName(this.GetType().Assembly.Location);
            }
            catch (Exception ex)
            {
                Debug.LogError("[KoreanPatch] 모드 폴더 경로를 찾을 수 없습니다: " + ex.Message);
            }
        }

        public void Start()
        {
            if (string.IsNullOrEmpty(modDirectoryPath)) return;

            LocalizationManager.OnSetLanguage += OnLanguageChanged;

            OnLanguageChanged(SystemLanguage.Korean);
        }

        public void OnDestroy()
        {
            LocalizationManager.OnSetLanguage -= OnLanguageChanged;
        }

        private void OnLanguageChanged(SystemLanguage newLanguage)
        {
            if (newLanguage == SystemLanguage.Korean)
            {
                LoadPatch();
            }
        }

        private void LoadPatch()
        {
            string csvPath = Path.Combine(modDirectoryPath, "Korean.csv");

            if (!File.Exists(csvPath))
            {
                Debug.LogWarning("[KoreanPatch] Korean.csv 파일을 찾을 수 없습니다");
                return;
            }

            try
            {
                var allLines = File.ReadAllLines(csvPath, Encoding.UTF8).Skip(1);
                int count = 0;

                foreach (string line in allLines)
                {
                    if (string.IsNullOrWhiteSpace(line)) continue;

                    int firstComma = line.IndexOf(',');
                    if (firstComma == -1) continue;

                    int lastComma = line.LastIndexOf(',');
                    int secondLastComma = line.LastIndexOf(',', lastComma - 1);

                    if (secondLastComma == -1 || lastComma == -1) continue;

                    string key = line.Substring(0, firstComma).Trim();
                    string value = line.Substring(firstComma + 1, secondLastComma - firstComma - 1).Trim();

                    if (value.StartsWith("\"") && value.EndsWith("\""))
                    {
                        value = value.Substring(1, value.Length - 2);
                    }

                    value = value.Replace("\\ ", " ");

                    value = value.Replace("\\n", "\n");

                    if (!string.IsNullOrEmpty(key))
                    {
                        LocalizationManager.SetOverrideText(key, value);
                        count++;
                    }
                }

                Debug.Log($"[KoreanPatch] {count}개의 한글 번역 텍스트 적용 완료");
            }
            catch (Exception ex)
            {
                Debug.LogError("[KoreanPatch] CSV 파일 읽기 오류: " + ex.Message);
            }
        }
    }
        }
