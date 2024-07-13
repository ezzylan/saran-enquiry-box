<script setup lang="ts">
import * as z from 'zod'
import isMobilePhone from 'validator/lib/isMobilePhone'
import { useFetch } from '@vueuse/core'

import { Button } from '@/components/ui/button'
import { AutoForm } from '@/components/ui/auto-form'
import { useToast } from '@/components/ui/toast/use-toast'
import { Toaster } from '@/components/ui/toast'

const { toast } = useToast()

const formSchema = z.object({
  name: z.string().min(3, {
    message: 'Name must be at least 3 characters.'
  }),
  mobileNumber: z.string().min(9).max(10).refine(isMobilePhone),
  email: z.string().email({ message: 'Invalid email address' }),
  message: z.string().describe('Type your message')
})

function onSubmit(values: Record<string, any>) {
  let telegramBotUrl = `https://api.telegram.org/bot${import.meta.env.VITE_TELEGRAM_BOT_TOKEN}/sendMessage`

  let bodyContent = new FormData()
  bodyContent.append('chat_id', '@saranenquiriestest')
  bodyContent.append(
    'text',
    `
    New enquiry!
    Name: ${values.name}
    Mobile Number: ${values.mobileNumber}
    Email: ${values.email}
    Message: ${values.message}
    `
  )

  const { error } = useFetch(telegramBotUrl).post(bodyContent)

  if (error.value)
    toast({
      title: 'Uh oh! Something went wrong.',
      description: 'There was a problem with your request.',
      variant: 'destructive'
    })
  else {
    toast({
      description: 'Your message has been sent successfully!'
    })
  }
}
</script>

<template>
  <main class="container mx-auto p-8">
    <AutoForm
      class="space-y-6"
      :schema="formSchema"
      :field-config="{ message: { component: 'textarea' } }"
      @submit="onSubmit"
    >
      <Button type="submit">Submit</Button>
    </AutoForm>
  </main>

  <Toaster />
</template>
